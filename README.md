

Vagrant chef solo berks starter
===============================


## Prerequisites ##

1. Make sure you already install vagrant.
2. Below vagrant plugins are also required.

        $ vagrant plugin install vagrant-cachier
        $ vagrant plugin install vagrant-omnibus
        $ vagrant plugin install vagrant-berkshelf
        
3. Make sure chef is installed
4. As well as knifo-solo, berkshelf

        $ gem install knife-solo
        $ gem install berkshelf


## Steps ##

1. Create your project directory.

        $ mkdir <my-project>
        $ cd <my-project>

2. Create `cookbooks` folder in you project folder, and go into it.

        $ mkdir cookbooks 
        $ cd cookbooks 

3. Create your cookbook.

        $ berks cookbook my-project my-project

4. Move the generated `Vagrantfile` to the project directory.

        $ mv Vagrantfile ../../

5. Modify the `Vagrantfile` if you like, such as change the vm box to "hashicorp/precise64".
   As you put your cookbook in `cookbooks`, you also have to specify the berkfile path.

        # The path to the Berksfile to use with Vagrant Berkshelf
        config.berkshelf.berksfile_path = "cookbooks/my-project/Berksfile"b


6. To reduce your apt installation time, backup the origin `sources.list` file, replace it with the faster repository. Create a `sources.list.erb` in `<my-project>/cookbooks/my-project/templates` folder.

        ```
        deb http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
        ```

7. Add dependency cookbook to your `metadata.rb`. For example, we use "apt" cookbook to perform apt update.

        ```
        name             'my-project'
        maintainer       'YOUR_NAME'
        maintainer_email 'YOUR_EMAIL'
        license          'All rights reserved'
        description      'Installs/Configures demo-project'
        long_description 'Installs/Configures demo-project'
        version          '0.1.0'

        depends 'apt', '~> 6.1.0'
        ```

8. Edit `recipes/default.rb`.

        ```ruby
        # update source.list
        # sudo mv /etc/apt/source.list /etc/apt/source.list.orig

        bash "backup origin sources.list" do
          code <<-EOL
          mv /etc/apt/sources.list /etc/apt/sources.list.orig 
          EOL
        end

        # chef generate template . source.list
        template '/etc/apt/sources.list' do
          source 'sources.list.erb'
        end


        # apt-update
        include_recipe 'apt::default'

        # install your favorite packages
        node['pkgs'].each do |pkg|
          package pkg
        end
        ```

8. In above recipe, we need to define `pkgs`. As a result, create `default.rb` in "attributes" folder.

        ```
        default['pkgs'] = %w(git emacs)
        ```

9. Goto our cookbook path, run `berks install` to install it.

10. Finally, our project directory looks as belows.

        .
        ├── README.md
        ├── Vagrantfile
        └── cookbooks
            └── my-project
                ├── Berksfile
                ├── Berksfile.lock
                ├── CHANGELOG.md
                ├── Gemfile
                ├── LICENSE
                ├── README.md
                ├── Thorfile
                ├── attributes
                │   └── default.rb
                ├── chefignore
                ├── files
                │   └── default
                ├── libraries
                ├── metadata.rb
                ├── providers
                ├── recipes
                │   └── default.rb
                ├── resources
                └── templates
                    ├── default
                    └── sources.list.erb


We are almost done. Go to top project directory, type `vagrant up` to bootstrap our fresh server.

