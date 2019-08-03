---
title: "Docker Logging With Rsyslog"
date: 2019-07-27T12:15:28+01:00
draft: false
lang: en-GB
tags: ["docker", "rsyslog", "syslog", "logging"]
---
<style type="text/css">
.hl {color: #f155f1;}
.hlb {color: #f155f1; font-weight: bold;}
</style>

## Introduction
This document will describe a simple strategy to logging for docker container using Rsyslog. Often we may have to run multiple containers on single machine. We may require logging for different container in different directories or files. This can be achieved using Rsyslog. Approach below is very generic and flexible and can be modified as per requirement easily.

## Running Docker with syslog logging
I use following command to run docker container.

<pre>
docker run --rm --log-driver=syslog  --log-opt tag=<span class="hl">"{{.ImageName}}/{{.Name}}/{{.ID}}"</span> ubuntu echo atestoutput
</pre>

Above command will append a line similar to following line in `/var/log/syslog` file.
```
Jun  7 15:58:27 machine docker/ubuntu/trusting_dubinsky/b7d1373fccaf[20642]: atestoutput
```
## What we want:
We want the logging of container to go in its own directory. E.g. We probably want that each docker container should log into `/var/log/company/dockerapps/containerName/docker.log`. For a few containers we may hard code into `rsyslog.conf` file. But for a number of containers, we will have to use more generic configuration of rsyslog. 

Take a look at pattern in above output i.e `docker/ubuntu/trusting_dubinsky/b7d1373fccaf[20642]`. This is called `syslog tag` and rsyslog will store this pattern in its property variable which can be accessed by referring to `syslogtag` in `rsyslog.conf` file. We will exploit this property to generate dynamic filenames and directory structure.

### Version of Rsyslog used
```bash
$ rsyslogd -v
        rsyslogd 8.16.0, compiled with:
        PLATFORM:                               x86_64-pc-linux-gnu
        PLATFORM (lsb_release -d):
        FEATURE_REGEXP:                         Yes
        GSSAPI Kerberos 5 support:              Yes
        FEATURE_DEBUG (debug build, slow code): No
        32bit Atomic operations supported:      Yes
        64bit Atomic operations supported:      Yes
        memory allocator:                       system default
        Runtime Instrumentation (slow code):    No
        uuid support:                           Yes
        Number of Bits in RainerScript integers: 64
```

### Configure rsyslog:
Create a regular file `/etc/rsyslog.d/40-dockerapp.conf`. We will put our configuration in this file. rsyslog comes with various defaults in `/etc/rsyslog.conf` and `/etc/rsyslog.d/50-default.conf`. We named our file as `40-dockerapp.conf`, because we want to get it executed before `50-default.conf`. Populate `/etc/rsyslog.d/40-dockerapp.conf` file with following contents.

```bash
# To create logging directories/filenames dynamically.
template(name="Dockerlogfiles" type="string" string="/var/log/company/dockerapps/%syslogtag:R,ERE,2,FIELD:docker/(.*)/(.*)/(.*)\\[--end%/%syslogtag:R,ERE,3,FIELD:docker/(.*)/(.*)/(.*)\\[--end%/docker.log")

# To format message that will be forwarded to remote syslog server
template (name="LongTagForwardFormat" type="string" string="<%PRI%>%TIMESTAMP:::date-rfc3339% %HOSTNAME% %syslogtag:::sp-if-no-1st-sp%%syslogtag%%msg:::sp-if-no-1st-sp%%msg%")

if $syslogtag startswith "docker" then {

    # Local logging
    action(name="localFiles" type="omfile" DynaFile="Dockerlogfiles")

    # Remote logging to remote syslog server. 
    action(name="forwardToOtherSyslog" type="omfwd" Target="IP ADDRESS of Target Syslog server" Port="514" Protocol="udp" template="LongTagForwardFormat")
    &~
}
```
Above template will generate filenames dynamically based on syslogtag pattern shown above in output. In this case we are using regular expressions. `&~` will stop the processing of further rules for the message. If `&~` is missing, then `50-default.conf` will pickup the message, process it and will send it to `/var/log/syslog` file as well. So we will eventually end up having duplicate syslog entries in two different files. 

#### Details of regular expressions used above:
_regex details_
```bash
%syslogtag:R,ERE,1,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> ImageName
%syslogtag:R,ERE,2,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> ContainerName
%syslogtag:R,ERE,3,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> ContainerID

Note: more accurately above statements are actually called property replacers using regex
```

##### NOTE1:
Rsyslog limits length of `syslogtag` to 32 characters when message is sent to remote rsyslog server(32 char limit is NOT there for local logging though).My tags are longer than 32 characters. Therefore I used another template that will not restrict the limit to 32 when sending the message to remote server. This template is taken from <http://www.rsyslog.com/sende-messages-with-tags-larger-than-32-characters/>.
However I have modified this template.

##### NOTE2:
I have modified the above template `"not to restrict syslogtag length limit to 32 chars"` a bit as stated above. The reason for doing this is very subtle. My syslogtag is greater than 32 characters and it consists of format like `"docker/A_very_long_docker_image_name:A_very_long_docker_tag_name/containerName/ContainerID:[somePID]"`. Take a careful look at `":"` colon in the middle and near the end of above `syslogtag`(before somePID). 
While forwarding above syslogtag (i.e message containing this syslogtag) to remote rsyslog server, destination rsyslog server was creating a `space` after the middle `:` colon. In order to prevent this I added `%syslogtag:::sp-if-no-1st-sp%` construct in above mentioned template. 

#### Restart rsyslog:
Restart rsyslog and test by starting container with command shown above. You will see a file docker.log in `/var/log/company/dockerapps/containerName/containerID/docker.log` 

Note:Rsyslog will create directory structure automatically.

#### Other ways:
Dynamic filename generation can be done in other ways too. Following are the other ways to write above template.

##### Using Latest template syntax, String type and Field values.
_template syntax_
```bash
# Using Latest template syntax (String type and Field values)
#        /%syslogtag:F,47:1% => represents docker in $syslogtag . 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:2% => represents ImageNAme. 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:3% => represents ContainerName.
#        /%syslogtag:F,47:4% => represents ContainerID.
template(name="Dockerlogfiles" type="string" string="/var/log/company/dockerapps/%syslogtag:F,47:2%/%syslogtag:F,47:3%/docker.log")
```

#### Using old template syntax and Using field values.
_old template syntax_
```bash
#        /%syslogtag:F,47:1% => represents "docker" in $syslogtag . 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:2% => represents ImageNAme. 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:3% => represents ContainerName.
#        /%syslogtag:F,47:4% => represents ContainerID.
# $template Dockerlogfiles, "/var/log/company/dockerapps/%syslogtag:F,47:2%/docker.log"
$template Dockerlogfiles, "/var/log/company/dockerapps/%syslogtag:F,47:2%/%syslogtag:F,47:3%/docker.log"
```

#### Using latest template syntax and list type
_Template syntax list type_
```bash
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="1") == will generate ==> imagename
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2") == will generate ==> containername
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2") == will generate ==> containerid
template(name="Dockerlogfiles" type="list") {
   constant(value="/var/log/company/dockerapps/")
   property(name="syslogtag" securepath="replace" \
            regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2")\
   constant(value="/docker.log")
}
```
##### A consolidated file describing all above scenarios is below:
_Full file_
```bash
# We assume that a syslogtag generated by docker container is of following format
#        docker/ImageNAme/ContainerName/ContainerID
# An exmaple docker command to generate above tag is
#        docker run --rm --log-driver=syslog  --log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}" ubuntu echo atestwithouttag
#    Above command will log following similar message
#        Jun  7 15:58:27 machine docker/ubuntu/trusting_dubinsky/b7d1373fccaf[20642]: atestwithouttag



# A very Simple way to filter messages
#:syslogtag, ereregex, "docker/ubuntu" /var/log/docker-syslog/syslog.log


# To create logging directories/filenames dynamically.
# Using latest template syntax (string type and regex). Below will create log file like : /var/log/company/dockerapps//docker.log
#     %syslogtag:R,ERE,1,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> imageName
#     %syslogtag:R,ERE,2,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> containerName
#     %syslogtag:R,ERE,3,FIELD:docker/(.*)/(.*)/(.*)\\[--end%  == will generate ==> containerID
# template(name="Dockerlogfiles" type="string" string="/var/log/company/dockerapps/%syslogtag:R,ERE,2,FIELD:docker/(.*)/(.*)/(.*)\\[--end%/docker.log")
template(name="Dockerlogfiles" type="string" string="/var/log/company/dockerapps/%syslogtag:R,ERE,2,FIELD:docker/(.*)/(.*)/(.*)\\[--end%/%syslogtag:R,ERE,3,FIELD:docker/(.*)/(.*)/(.*)\\[--end%/docker.log")

# To format message that will be forwarded to remote syslog server
template (name="LongTagForwardFormat" type="string" string="<%PRI%>%TIMESTAMP:::date-rfc3339% %HOSTNAME% %syslogtag:::sp-if-no-1st-sp%%syslogtag%%msg:::sp-if-no-1st-sp%%msg%")

# Using Latest template syntax (String type and Field values)
#        /%syslogtag:F,47:1% => represents docker in $syslogtag . 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:2% => represents ImageNAme. 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:3% => represents ContainerName.
#        /%syslogtag:F,47:4% => represents ContainerID.
# template(name="Dockerlogfiles" type="string" string="/var/log/company/dockerapps/%syslogtag:F,47:2%/%syslogtag:F,47:3%/docker.log")



# Using old template syntax and Using field values.
#        /%syslogtag:F,47:1% => represents "docker" in $syslogtag . 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:2% => represents ImageNAme. 47 is ASCII decimal value of / character
#        /%syslogtag:F,47:3% => represents ContainerName.
#        /%syslogtag:F,47:4% => represents ContainerID.
# $template Dockerlogfiles, "/var/log/company/dockerapps/%syslogtag:F,47:2%/docker.log"
# $template Dockerlogfiles, "/var/log/company/dockerapps/%syslogtag:F,47:2%/%syslogtag:F,47:3%/docker.log"




# Using latest template method (list type)
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="1") == will generate ==> imageName
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2") == will generate ==> containerName
#     regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2") == will generate ==> containerID
#template(name="Dockerlogfiles" type="list") {
#   constant(value="/var/log/company/dockerapps/")
#   property(name="syslogtag" securepath="replace" \
#            regex.expression="docker/\\(.*\\)/\\(.*\\)/\\(.*\\)\\[" regex.submatch="2")\
#   constant(value="/docker.log")
#
#}

if $syslogtag startswith "docker" then {
    # Local logging
    action(name="localFiles" type="omfile" DynaFile="Dockerlogfiles")

    # Remote logging to remote syslog server. 
    action(name="forwardToOtherSyslog" type="omfwd" Target="IP ADDRESS of Target Syslog server" Port="514" Protocol="udp" template="LongTagForwardFormat")
    &~
}
```


