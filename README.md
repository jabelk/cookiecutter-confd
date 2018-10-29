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
ntc@ntc:githubtests$ cookiecutter /home/ntc/cookiecutter-confd
project_name [my-awesome-confd-project]:
project_description [baseline ConfD Project for NTC]:
author_name [Jason Belk]:
author_email [jason.belk@networktocode.com]:
yang_module [test_yang]:
ntc@ntc:githubtests$ ls
my-awesome-confd-project  ntc_confd_project
ntc@ntc:githubtests$ cd ntc_confd_project/
ntc@ntc:ntc_confd_project$ cd ..
ntc@ntc:githubtests$ cd my-awesome-confd-project/
ntc@ntc:my-awesome-confd-project$ ls
action.py               cmd-invoke-action-err.xml  confd.conf  Makefile            README.md
cmd-invoke-action2.xml  cmd-invoke-action.xml      dynamo.py   Makefile.inc.basic  test_yang.yang
```

### Now start the confd daemon

```
ntc@ntc:my-awesome-confd-project$ make all
/home/ntc/confd-install/bin/confdc --fail-on-warnings  -c -o test_yang.fxs  test_yang.yang
/home/ntc/confd-install/bin/confdc  --emit-python test_yang_ns.py test_yang.fxs
mkdir -p ./confd-cdb
cp /home/ntc/confd-install/var/confd/cdb/aaa_init.xml ./confd-cdb
ln -s /home/ntc/confd-install/etc/confd/ssh ssh-keydir
Build complete
ntc@ntc:my-awesome-confd-project$ make start
### Killing any confd daemon and confd agents
/home/ntc/confd-install/bin/confd --stop || true
connection refused (stop)
killall `pgrep -f "python ./action.py"` || true
6987: no process found
/home/ntc/confd-install/bin/confd -c confd.conf --addloadpath /home/ntc/confd-install/etc/confd
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
ntc@ntc:~$ kickoffconfd

admin connected from 10.0.2.2 using ssh on ntc
ntc# conf
Entering configuration mode terminal
ntc(config)# test_action dynamo_rest_call
Possible completions:
  <cr>
ntc(config)# test_action dynamo_rest_call
rest_output Result from REST Call

{
"cisco"   : {
    "hosts"   : [ "n9k1.ntc.com", "n9k2.ntc.com" ],
    "vars"    : {
        "platform"   : "nexus"
    }
},
"arista"  : [ "arista1.ntc.com", "arista2.ntc.com" ],
"hp"     : {
    "hosts"   : [ "hp1.ntc.com", "hp2.ntc.com", "hp3.ntc.com", "hp4.ntc.com" ],
    "vars"    : {
        "platform"   : "comware7"
    }
},
"juniper"    : [ "jnprfw.ntc.com" ],
"apic"     : [ "aci.ntc.com" ]
}


ntc(config)# test_action dynamo_rest_call |
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
ntc(config)# test_action dynamo_rest_call | display xp
                                                    ^
% Invalid input detected at '^' marker.
ntc(config)# test_action dynamo_rest_call | display ?
Possible completions:
  json   Display output as json
  xml    Display output as XML
ntc(config)# test_action dynamo_rest_call | display xml
<rest_output xmlns='http://tail-f.com/ns/example/test_yang'>Result from REST Call

{&#13;
"cisco"   : {&#13;
    "hosts"   : [ "n9k1.ntc.com", "n9k2.ntc.com" ],&#13;
    "vars"    : {&#13;
        "platform"   : "nexus"&#13;
    }&#13;
},&#13;
"arista"  : [ "arista1.ntc.com", "arista2.ntc.com" ],&#13;
"hp"     : {&#13;
    "hosts"   : [ "hp1.ntc.com", "hp2.ntc.com", "hp3.ntc.com", "hp4.ntc.com" ],&#13;
    "vars"    : {&#13;
        "platform"   : "comware7"&#13;
    }&#13;
},&#13;
"juniper"    : [ "jnprfw.ntc.com" ],&#13;
"apic"     : [ "aci.ntc.com" ]&#13;
}

</rest_output>

ntc(config)# exit
ntc# exit
```

