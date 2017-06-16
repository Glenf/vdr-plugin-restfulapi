# RESTFULAPI-Plugin for VDR

## Table Of Contents

1. Preface
1. Requirements
1. Configuration
1. Channels
    1. Groups
    1. Images / Channel-Logos
1. Events / EPG
    1. Images
    1. Search (EPGSEARCH)
    1. Get Contentdescriptors
1. Info
1. Osd
1. Recordings
    1. Cut
    1. Marks
    1. Play
1. Remote
1. Timers
    1. Reading/Deleting Timers
    1. Bulk Delete Timers
    1. Creating Timers
    1. Updating Timers
1. SearchTimers (EPGSEARCH)
    1. Create
    1. Delete
    1. Search
    1. Get blacklists
    1. Get recording dirs
    1. Get channel groups
    1. Fetch ext EPG info (epgsearchcats.conf)
    1. Check timer conflicts
1. ScraperImages
1. Control the Wirbelscan plugin
    1. Get the current state
    1. Get the list of countries
    1. Get the list of satellites
    1. Get the current setup
    1. Changes the current setup
    1. Launches a command
1. Webapp
1. Femon
1. JSDoc

## Preface

This plugin has been developed to offer a modern API for other developers to communicate with the VDR.

The plugin supports the following outputs formats: xml, json and html.
It also supports the following input formats: json and html.

It would not exist without the help of the following people:

* Klaus Schmidinger: Of course because of the VDR itself. Thank you also for the nice and clean layout for the API-Documentation which I "borrowed" from your PLUGINS.html.
* Gerald Dachs: Thank you for the intial idea and the initial jsonapi-Plugin.
* Volker Richert: Thank you for fixing the UTF8-Support.
* Tommi Mäkitalo: Thank you for implementing the Regex-Support in cxxtools which allows RESTful web services.
* Holger Schvestka: Thank you for improving the package structure.
* All other yaVDR-Members who I haven't already mentioned, just because you are who you are and do your best to improve the VDR experience.
* The developers of the live and vnsisserver Plugins which helped me to learn a lot about the API of VDR.
* The developers of the live plugin where I borrowed parts of the epgsearch-implementation :-)
* Daniel Kuschny: Thank you for fixing my broken regular expressions. Will pay you a beer! :-)
* Keine_Ahnung@vdr-portal for the VDR 1.6 compatibility patch

**This document describes the API and how to use it. It contains a lot of examples for the different formats and services but there is still a lot more to discover if you simply try to make a few simple requests.**

## Requirements
Someone who wants to install the plugin on his/her VDR needs following applications and libraries:

* VDR 1.7.18
* libcxxtools Rev. >= 1231, which is available as package for Ubuntu in the yavdr-PPA's

Someone who wants to develop an application which uses this API:

* XML or JSON Parser (Depends on which format you want to use! - You can also use the html-format, but that one is more a proof for the restful concept and does not show all information!)
* URL Decoder or JSON Serializer (to send data to the webservice, required e.g. to create timers, searchtimers etc...) e.g. POSTMan REST Client. There is a wide variety of REST clients available on the web.

## Configuration

Create a new file called plugin.restfulapi.conf in /etc/vdr/plugins/ respectively /etc/vdr/conf.avail (don't forget to create a symlink in /etc/vdr/conf.d/).

```bash
#Command line parameters for vdr-plugin-restfulapi #
--port=8002 --ip=0.0.0.0 \
--epgimages=/var/cache/vdr/epgimages \
--channellogos=/usr/share/vdr/channel-logos \
--webapp=/var/lib/vdr/plugins/restfulapi/webapp,/var/lib/vdr/plugins/restfulapi/a-second-webapp
```

## Channels
This service returns a list of channels.

** Examples **

| Method | URL |
|-|-|
| GET | ```http://<ip>:<port>/channels.<format>``` |
| GET | ```http://<ip>:<port>/channels/<channelid>.<format>``` |
| GET | ```http://<ip>:<port>/channels.format?start=<start>&limit=<limit>``` |

| Param | Description |
|-|-|
| format | json, xml or html |
| channelid | Id of the requested channel |
| start | Start is the first element you want to retrieve |
| limit | Limit... |

** XML response **

```xml
GET http://127.0.0.1:8002/channels.xml?start=0&limit=1

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <channels xmlns="http://www.domain.org/restfulapi/2011/channels-xml">
        <channel>
            <param name="name">ORF1 HD</param>
            <param name="number">1</param>
            <param name="channel_id">C-71-71-61920</param>
            <param name="image">true</param>
            <param name="group">hd</param>
            <param name="transponder">330</param>
            <param name="stream">C-71-71-61920.ts</param>
            <param name="is_atsc">false</param>
            <param name="is_cable">true</param>
            <param name="is_sat">false</param>
            <param name="is_terr">false</param>
            <param name="is_radio">false</param>
        </channel>
        <count>1</count>
        <total>259</total>
</channels>
```
** JSON response **

```json
GET http://127.0.0.1:8002/channels.json?start=0&limit=1
{
    "channels": [
        {
            "name":"ORF1 HD",
            "number":1,
            "channel_id":"C-71-71-61920",
            "image":true,
            "group":"hd",
            "transponder":330,
            "stream":"C-71-71-61920.ts",
            "is_atsc":false,
            "is_cable":true,
            "is_terr":false,
            "is_sat":false,
            "is_radio": false
        }
    ],
    "count":1,
    "total":259
}
```

### Groups

This service will return a list containing all channel groups.

| Method | URL |
|-|-|
| GET | ```http://<ip>:<port>/channels/groups.<format>```|
| GET | ```http://<ip>:<port>/channels/groups.<format>?start=<start>&limit=<limit>```|
| GET | ```http://<ip>:<port>/channels.<format>?group=<group>```|

| Param | Description |
|-|-|
| format | json, xml or html |
| group | returns the channels of the requested group |

** XML response **
```xml
GET http://<ip>:<port>/channels/groups.xml?start=0&limit=3
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<groups xmlns="http://www.domain.org/restfulapi/2011/groups-xml">
    <group>hd</group>
    <group>main</group>
    <group>stuff</group>
    <count>3</count>
    <total>3</total>
</groups>
```

** JSON response **

```json
GET http://<ip>:<port>/channels/groups.json?start=0&limit=3
{
    "groups": [
        "hd",
        "main",
        "stuff"
    ],
    "count":3,
    "total":3
}
```
