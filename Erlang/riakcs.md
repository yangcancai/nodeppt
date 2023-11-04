# 说明
- create bucket只能使用s3cmd
- s3cmd本机和外网创建配置不一样
- nginx签名路径要跟签名一致

# s3cmd本地配置

```shell
[default]
access_key = <YOUR KEY>
bucket_location = US
cloudfront_host = cloudfront.amazonaws.com
cloudfront_resource = /2010-07-15/distribution
default_mime_type = binary/octet-stream
delete_removed = False
dry_run = False
enable_multipart = False
encoding = UTF-8
encrypt = False
follow_symlinks = False
force = False
get_continue = False
gpg_command = /usr/local/bin/gpg
gpg_decrypt = %(gpg_command)s -d --verbose --no-use-agent --batch --yes --passphrase-fd %(passphrase_fd)s -o %(output_file)s %(input_file)s
gpg_encrypt = %(gpg_command)s -c --verbose --no-use-agent --batch --yes --passphrase-fd %(passphrase_fd)s -o %(output_file)s %(input_file)s
gpg_passphrase = password
guess_mime_type = True
host_base = s3.amazonaws.com
host_bucket = %(bucket)s.s3.amazonaws.com
human_readable_sizes = False
list_md5 = False
log_target_prefix =
preserve_attrs = True
progress_meter = True
# 这里需要配置为内网ip
proxy_host = localhost
proxy_port = 8080
recursive = False
recv_chunk = 4096
reduced_redundancy = False
secret_key = <YOUR SECRET> 
send_chunk = 4096
simpledb_host = sdb.amazonaws.com
skip_existing = False
socket_timeout = 300
urlencoding_mode = normal
use_https = False
verbosity = WARNING
signature_v2 = True
```

# s3cmd外网配置

```shell
[default]
access_key = <YOUE KEY> 
bucket_location = US
cloudfront_host = cloudfront.amazonaws.com
cloudfront_resource = /2010-07-15/distribution
default_mime_type = binary/octet-stream
delete_removed = False
dry_run = False
enable_multipart = False
encoding = UTF-8
encrypt = False
follow_symlinks = False
force = False
get_continue = False
gpg_command = /usr/local/bin/gpg
gpg_decrypt = %(gpg_command)s -d --verbose --no-use-agent --batch --yes --passphrase-fd %(passphrase_fd)s -o %(output_file)s %(input_file)s
gpg_encrypt = %(gpg_command)s -c --verbose --no-use-agent --batch --yes --passphrase-fd %(passphrase_fd)s -o %(output_file)s %(input_file)s
gpg_passphrase = password
guess_mime_type = True
host_base = <YOUR DOMAIN HERE>
host_bucket = %(bucket)s.<YOUR DOMAIN HERE>
human_readable_sizes = False
list_md5 = False
log_target_prefix =
preserve_attrs = True
progress_meter = True
proxy_host =
proxy_port = 0
recursive = False
recv_chunk = 4096
reduced_redundancy = False
secret_key = <YOUR SECRET> 
send_chunk = 4096
simpledb_host = sdb.amazonaws.com
skip_existing = False
socket_timeout = 300
urlencoding_mode = normal
use_https = True
verbosity = WARNING
signature_v2 = True
```
# openrestry主配置

```lua
upstream riakcs_hosts {
    server  192.168.3.22:8080;
}
...
location @csproxy {
        -- 注意这里需要使用bucket名字作为域名的前缀
        -- <BUCKET>.user.com
        proxy_set_header  Host test-abc.user.com;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto http;
        proxy_set_header  X-Forwarded-For $remote_addr;
        proxy_pass    http://riakcs_hosts;
    }
    ...
-- /<BUCKET>/<KEY>
-- /test-abc/user.jpg
 location /test-abc {
        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Credentials true;
            add_header Access-Control-Allow-Methods 'GET, PUT, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Authorization,DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
            return 200;
        }
        client_max_body_size 60M;
        access_by_lua_file cs_auth.lua;
        -- 注意这里要rewrite test-abc桶
        -- rewrite /test-abc/(.*) /test-abc/$1 break;
        try_files $uri $uri/ @csproxy;
    }
 ```
# accept_lua_file脚本

```lua
--- description:
-- 1 check jwt_token
-- 2 generate cs signature

local m = ngx.req.get_method()
-- POST和PUT统一修改为PUT方法
if ngx.req.get_method() == "POST" then
    m = "PUT"
    ngx.req.set_method(ngx.HTTP_PUT)
end
local h = ngx.req.get_headers()
local md5 = h["content-md5"]
local content_type = h["content-type"]
if content_type == nil then 
    content_type = ""
end
local timestamp = h["timestamp"]
local file_type = h["file-type"]
local request_uri = ngx.var.request_uri
local request_host = ngx.var.host
-- Fix me: check jwt_token.
local m = ngx.req.get_method()
local h = ngx.req.get_headers()
-- token放到Authorization或者路径参数
local auth = h["Authorization"]
if (auth == nil or auth == "") then
   local args, err = ngx.req.get_uri_args()
   auth = ""
   auth = args["token"]
end
if (nil == auth) or ("" == auth) then
	ngx.log(ngx.ERR, "no authorization")
	--ngx.exit(ngx.HTTP_UNAUTHORIZED)
	ngx.exit(ngx.HTTP_FORBIDDEN)
end


local cjson = require "cjson"
local jwt = require "resty.jwt"

local secret = "<YOUR JWT SECRET>"

local jwt_obj = jwt:verify(secret, auth)

if jwt_obj.verified == false then
--	ngx.log(ngx.ERR, "Invalid token: ".. jwt_obj.reason..",auth:"..auth)
	ngx.exit(ngx.HTTP_FORBIDDEN)
end
-- generate cs signature.

local now_seconds = ngx:time()
local date = ngx.http_time(now_seconds)
if (nil == md5)
then
	ngx.log(ngx.NOTICE, "md5 is empty end")
	md5 = ""
end
-- 去掉路径参数
local uri_without_params = string.match(request_uri, "^[^?]*")
local stringtosign = m.."\n"..md5.."\n"..content_type.."\n"..date.."\n"..uri_without_params
local key="<YOUR riakcs KEY>"
local access_key = "<YOUR riakcs SECRET KEY>"
local digest = ngx.hmac_sha1(access_key, stringtosign)
local encode = ngx.encode_base64(digest)
local auth = "AWS "..key..":"..encode

ngx.log(ngx.ERR, stringtosign)
ngx.log(ngx.ERR, request_uri)
ngx.req.set_header("date", date)
if content_type ~= "" then
    ngx.req.set_header("content-type", content_type)
end
ngx.req.set_header("Authorization", auth)

```