# Chef

[Back to guide index](README.md)

Chef is a configuration management platform that uses cookbooks and recipes to define system state and operational behavior.

Chef has historically used a client-server architecture, though local modes also exist.

## 5.1 Chef Architecture

| Component | Description |
|---|---|
| Workstation | Where cookbooks are authored |
| Chef Server | Stores cookbooks, node data, policies |
| Chef Client | Runs on nodes and applies recipes |
| Cookbook | Collection of recipes and files |
| Recipe | Ruby-based configuration instructions |
| Resource | Declarative unit like package or service |
| Attribute | Node data for configuration |
| Data Bag | Secure or structured data storage |

## 5.2 Basic Workflow

1. Author cookbooks on a workstation.
2. Upload cookbooks and policies to Chef Server.
3. Chef Client runs on nodes.
4. Node converges according to assigned roles or policyfiles.

## 5.3 Cookbook Structure

```text
cookbooks/
└── webserver/
    ├── attributes/
    │   └── default.rb
    ├── files/
    ├── recipes/
    │   └── default.rb
    ├── templates/
    ├── metadata.rb
    └── README.md
```

## 5.4 Resources in Chef

Common resources include:

- package
- service
- file
- template
- user
- group
- directory
- execute
- cron

## 5.5 Basic Recipe Example

```ruby
package 'nginx' do
  action :install
end

directory '/var/www/html' do
  owner 'root'
  group 'root'
  mode '0755'
  action :create
end

file '/var/www/html/index.html' do
  content "Managed by Chef\n"
  mode '0644'
  action :create
end

service 'nginx' do
  action [:enable, :start]
end
```

## 5.6 Attributes

Attributes provide configurable values.

### attributes/default.rb

```ruby
default['webserver']['port'] = 80
default['webserver']['server_name'] = 'localhost'
```

### Using attributes in a template

```ruby
template '/etc/nginx/conf.d/site.conf' do
  source 'site.conf.erb'
  variables(
    port: node['webserver']['port'],
    server_name: node['webserver']['server_name']
  )
  notifies :restart, 'service[nginx]', :delayed
end
```

## 5.7 Template Example

### templates/default/site.conf.erb

```erb
server {
  listen <%= @port %>;
  server_name <%= @server_name %>;
}
```

## 5.8 Notifications

Chef resources can notify other resources.

```ruby
service 'nginx' do
  action [:enable, :start]
end

file '/etc/nginx/nginx.conf' do
  content 'worker_processes auto;'
  notifies :restart, 'service[nginx]', :delayed
end
```

## 5.9 Roles

Roles can group recipes and attributes for node types.

```ruby
name 'webserver'
description 'Role for web nodes'
run_list 'recipe[webserver]'
default_attributes(
  'webserver' => {
    'port' => 80
  }
)
```

## 5.10 Data Bags

Data bags store structured data.

Examples:

- User account lists
- Application secrets
- Environment-specific settings

Encrypted data bags can protect sensitive content.

## 5.11 Chef Supermarket

Chef Supermarket is a repository of cookbooks.

Guidelines:

- Review community cookbooks carefully.
- Pin versions.
- Wrap generic cookbooks with internal policy.

## 5.12 Policyfiles

Policyfiles are a modern way to define cookbook sets and versions.

They can reduce some of the complexity of roles and environments.

## 5.13 Example: User Management Recipe

```ruby
group 'adminops' do
  action :create
end

user 'deploy' do
  manage_home true
  shell '/bin/bash'
  group 'adminops'
  action :create
end
```

## 5.14 Example: Security Hardening Recipe

```ruby
package 'openssh-server' do
  action :install
end

ruby_block 'disable_root_ssh' do
  block do
    file = Chef::Util::FileEdit.new('/etc/ssh/sshd_config')
    file.search_file_replace_line(/^PermitRootLogin/, 'PermitRootLogin no')
    file.write_file
  end
  notifies :restart, 'service[sshd]', :delayed
end

service 'sshd' do
  action [:enable, :start]
end
```

## 5.15 Example: Package Baseline

```ruby
%w(curl vim git rsync).each do |pkg|
  package pkg do
    action :install
  end
end
```

## 5.16 Test Kitchen

Chef ecosystems often use Test Kitchen for testing cookbooks in isolated environments.

## 5.17 InSpec

InSpec is commonly used for compliance and infrastructure validation.

Example test concepts:

- Check package installed
- Check service enabled
- Check config file contents
- Check ports listening

## 5.18 Best Practices for Chef

1. Prefer declarative resources over raw execute.
2. Use Policyfiles where appropriate.
3. Test cookbooks with Test Kitchen.
4. Store secrets securely.
5. Use templates and attributes for reuse.
6. Keep recipes focused.
7. Use notifications for controlled service restarts.

## 5.19 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| Overusing ruby_block and execute | Harder to reason about | Prefer native resources |
| Attribute sprawl | Hard to manage | Use a clear attribute strategy |
| Large monolithic cookbooks | Low reuse | Split by concern |
| No test coverage | Higher drift risk | Use Kitchen and InSpec |

## 5.20 When Chef Fits Well

Chef is a strong fit when teams are comfortable with Ruby-based DSLs and want rich configuration modeling with strong testing practices.

## 5.21 Chef Summary

Chef provides flexible, code-centric configuration management for Linux systems, especially where cookbook ecosystems and compliance testing are important.

---
