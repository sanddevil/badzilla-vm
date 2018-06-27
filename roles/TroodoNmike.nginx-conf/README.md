Install:

    ansible-galaxy install TroodoNmike.nginx-conf
    
Defaults:

    nginx_conf_default_path: /etc/nginx/conf.d
    nginx_conf_default_template: symfony2.conf


Playbook example:

    -
        role: TroodoNmike.nginx-conf
        nginx_conf_sites:
            web1:
              file_name: sites-external-api.conf       # field required
              template_params:
                server_name: external-api.mlp.dev      # field required
                root: /var/www/external-api/web        # field required
            web2:
              file_name: sites-another.conf       
              template_params:
                server_name: another.domain.com   
                root: /var/www/another/web        
            
You can add your own template very easy:

Create file in folder templates like this: 

        templates/template-your-name.conf.j2
        
Then simply add in config:

    -
        role: TroodoNmike.nginx-conf
        nginx_conf_sites:
            web1:
              file_name: sites-your-name.conf
              template_name: your-name.conf.j2        # optional and can be defined by the user
              template_params:
                server_name: domain.domain.com        # copy syntax from other template if you need params
                root: /var/www/yout-name/web          # copy syntax from other template if you need params

Optionally you can define basic auth like this:

    -
        role: TroodoNmike.nginx-conf
        nginx_conf_sites:
            web1:
              file_name: sites-your-name.conf
              template_name: your-name.conf.j2        # optional and can be defined by the user
              template_params:
                server_name: domain.domain.com        # copy syntax from other template if you need params
                root: /var/www/yout-name/web          # copy syntax from other template if you need params
                basic_auth:
                  username: test
                  password: test2
                  file_path: /etc/nginx/conf.d/.htpasswd-my-pass
