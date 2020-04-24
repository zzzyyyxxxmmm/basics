
# Understand puppet resource 

## exec 
```sh
exec { 'install-cat-picture-generator':
cwd => '/tmp/cat-picture-generator',
command => '/tmp/cat-picture/generator/configure && /usr/bin/make
install',
creates => '/usr/local/bin/cat-picture-generator',
}
```

The **cwd** attribute sets the working directory where the command will be run (current working directory). When installing software, this is generally the software source directory.

The **creates** attribute specifies a file which should exist after the command has been run. If this file is present, Puppet will not run the command again. This is very useful because without a creates attribute, an exec resource will run every time Puppet runs, which is generally not what you want. The creates attribute tells Puppet, in effect, "Run the exec only if this file doesn't exist."