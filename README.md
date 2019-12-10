# cookiecutter-confd

Inspired by Kevin Corbin's CookieCutter usage.

A template for ConfD projects using [cookiecutter](https://github.com/audreyr/cookiecutter)

## **Assumes using ConfD Premium for REST API test**

# How to use this template

```
    $ pip install cookiecutter
    $ cookiecutter https://github.com/jabelk/cookiecutter-confd.git
```

You will be asked about your basic info (name, project name, app name, etc.). This info will be used in your new project.

# Sample usage

Assuming the directory is git cloned already and python dependencies installed,
and bash profile updated `# Add ~/.local/ to PATH
export PATH=$HOME/.local/bin:$PATH`

## First 
```
cisco@cisco:githubtests$ cookiecutter /home/cisco/cookiecutter-confd
project_name [my-awesome-confd-project]:
project_description [baseline ConfD Project]:
author_name [Your Name Here]: Jason Belk
author_email [youremail@domain.com]: jabelk@cisco.com
yang_module [test_yang]:
cisco@cisco:githubtests$ ls
my-awesome-confd-project  cisco_confd_project
cisco@cisco:githubtests$ cd cisco_confd_project/
cisco@cisco:cisco_confd_project$ cd ..
cisco@cisco:githubtests$ cd my-awesome-confd-project/
cisco@cisco:my-awesome-confd-project$ ls
action.py               cmd-invoke-action-err.xml  confd.conf  Makefile            README.md
cmd-invoke-action2.xml  cmd-invoke-action.xml      dynamo.py   Makefile.inc.basic  test_yang.yang
```

### Now start the confd daemon

```
cisco@cisco:my-awesome-confd-project$ make all
/home/cisco/confd-install/bin/confdc --fail-on-warnings  -c -o test_yang.fxs  test_yang.yang
/home/cisco/confd-install/bin/confdc  --emit-python test_yang_ns.py test_yang.fxs
mkdir -p ./confd-cdb
cp /home/cisco/confd-install/var/confd/cdb/aaa_init.xml ./confd-cdb
ln -s /home/cisco/confd-install/etc/confd/ssh ssh-keydir
Build complete
cisco@cisco:my-awesome-confd-project$ make start
### Killing any confd daemon and confd agents
/home/cisco/confd-install/bin/confd --stop || true
connection refused (stop)
killall `pgrep -f "python ./action.py"` || true
6987: no process found
/home/cisco/confd-install/bin/confd -c confd.conf --addloadpath /home/cisco/confd-install/etc/confd
### * In one terminal window, run: tail -f ./confd.log
### * In another terminal window, run queries
###   (try 'make query' for an example)
### * In this window, the actions confd daemon now starts:
python action.py
info: 05-Sep-2018::05:51:28.000 - --- Daemon myaction STARTED ---
Hit <ENTER> to quit
info: 05-Sep-2018::05:52:04.000 - Making REST CALL
info: 05-Sep-2018::05:52:19.000 - Making REST CALL

```

### in another session access confd while daemon runs

assuming  `alias kickoffconfd
alias kickoffconfd='confd_cli -C -u admin'`

```
cisco@cisco:~$ kickoffconfd

admin connected from 10.0.2.2 using ssh on cisco
cisco# conf
Entering configuration mode terminal
cisco(config)# test_action dynamo_rest_call
Possible completions:
  <cr>
cisco(config)# test_action dynamo_rest_call
rest_output Result from REST Call

{
"cisco"   : {
    "hosts"   : [ "n9k1.cisco.com", "n9k2.cisco.com" ],
    "vars"    : {
        "platform"   : "nexus"
    }
},
"arista"  : [ "arista1.cisco.com", "arista2.cisco.com" ],
"hp"     : {
    "hosts"   : [ "hp1.cisco.com", "hp2.cisco.com", "hp3.cisco.com", "hp4.cisco.com" ],
    "vars"    : {
        "platform"   : "comware7"
    }
},
"juniper"    : [ "jnprfw.cisco.com" ],
"apic"     : [ "aci.cisco.com" ]
}


cisco(config)# test_action dynamo_rest_call |
Possible completions:
  append    Append output text to a file
  begin     Begin with the line that matches
  count     Count the number of lines in the output
  details   Display default values
  display   Display options
  exclude   Exclude lines that match
  include   Include lines that match
  linnum    Enumerate lines in the output
  more      Paginate output
  nomore    Suppress pagination
  save      Save output text to a file
  until     End with the line that matches
cisco(config)# test_action dynamo_rest_call | display xp
                                                    ^
% Invalid input detected at '^' marker.
cisco(config)# test_action dynamo_rest_call | display ?
Possible completions:
  json   Display output as json
  xml    Display output as XML
cisco(config)# test_action dynamo_rest_call | display xml
<rest_output xmlns='http://tail-f.com/ns/example/test_yang'>Result from REST Call

{&#13;
"cisco"   : {&#13;
    "hosts"   : [ "n9k1.cisco.com", "n9k2.cisco.com" ],&#13;
    "vars"    : {&#13;
        "platform"   : "nexus"&#13;
    }&#13;
},&#13;
"arista"  : [ "arista1.cisco.com", "arista2.cisco.com" ],&#13;
"hp"     : {&#13;
    "hosts"   : [ "hp1.cisco.com", "hp2.cisco.com", "hp3.cisco.com", "hp4.cisco.com" ],&#13;
    "vars"    : {&#13;
        "platform"   : "comware7"&#13;
    }&#13;
},&#13;
"juniper"    : [ "jnprfw.cisco.com" ],&#13;
"apic"     : [ "aci.cisco.com" ]&#13;
}

</rest_output>

cisco(config)# exit
cisco# exit
```

