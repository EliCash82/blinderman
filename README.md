# BLINDERMAN

### Middleman + NginX + Blinder.js + Heroku

This is an attempt to experiment with different middleman configurations for an optimal deployment
of static sites with jquery/javascript plugins.  I've chosen Blinder.js as an initial plugin for
testing.

---

### PART 1 
#### NginX Configuration

NginX Configuration from [RandomErrata](http://www.randomerrata.com/articles/2013/nginx-heart-middleman/):

1. Configure Heroku use of multi-buildpack (w/ NginX & Ruby buildpacks)

```

heroku config:set BUILDPACK_URL='https://github.com/ddollar/heroku-buildpack-multi.git'
echo 'https://github.com/ryandotsmith/nginx-buildpack.git' >> .buildpacks
echo 'https://codon-buildpacks.s3.amazonaws.com/buildpacks/heroku/ruby.tgz' >> .buildpacks
git add -f .buildpacks
git commit -m 'Add multi-buildpack'

```

2. Create `config` directory.  In `config` directory create file `nginx.conf.erb` with the following:

```

daemon off;
# Heroku dynos have 4 cores.
worker_processes 4;


events {
  use epoll;
  accept_mutex on;
  worker_connections 1024;
}


http {
  gzip on;
  gzip_comp_level 2;
  gzip_min_length 512;


  log_format l2met 'measure#nginx.service=$request_time request_id=$http_heroku_request_id';
  access_log logs/nginx/access.log l2met;
  error_log logs/nginx/error.log;


  include mime.types;
  default_type application/octet-stream;
  sendfile on;


  # Must read the body in 5 seconds.
  client_body_timeout 5;


  server {
    listen <%= ENV["PORT"] %>;
    server_name _;
    keepalive_timeout 5;
    root <%= ENV["HOME"] + "/build" %>;
  }
}

```

3. Create Procfile, insert following content:

```

web: bundle exec middleman build && bin/start-nginx -f sleep infinity

```

git commit and push and you've got a working NginX Middleman instance running on Heroku

---

