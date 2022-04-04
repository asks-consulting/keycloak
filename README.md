# keycloak

It has become time to replace `auth0.com` backed by Google Apps with
something else. Reasons `auth0` is no longer viable:

+ auth0.com no longer offers the free plan I originally signed up for, and 
  although it still works it no longer allows adding new users
+ auth0.com authentication having an annoyingly short lifetime, causing
  incessant logouts and sometimes loss of data in forms and such
+ [Google Suite free tier shutting down](https://arstechnica.com/gadgets/2022/01/google-tells-free-g-suite-users-pay-up-or-lose-your-account/)


I plan to start by simply maintaining a "list" of users/passwords in Keycloak
itself. Furthermore, I want to install Keycloak on its own LXC container
configured as far as possible for production, but without going so far as to
run multiple Keycloak instances with a shared database backend.


### Ansible roles

+ https://github.com/ansible-middleware/keycloak
+ https://github.com/andrewrothstein/ansible-keycloak
+ https://github.com/andrelohmann/ansible-role-keycloak
+ https://github.com/zhangran1/ansible-role-keycloak


### Ansible plugins

+ [community.general.keycloak_realm](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_realm_module.html)
+ [community.general.keycloak_role](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_role_module.html)
+ [community.general.keycloak_client](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_client_module.html)
+ [community.general.keycloak_authentication](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_authentication_module.html)
+ [community.general.keycloak_clienttemplate](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_clienttemplate_module.html)
+ [community.general.keycloak_client_rolemapping](https://docs.ansible.com/ansible/latest/collections/community/general/keycloak_client_rolemapping_module.html)
