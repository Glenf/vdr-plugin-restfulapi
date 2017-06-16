# RESTFULAPI-Plugin for VDR

Version 0.2.3.7

Copyright © 2015 yavdr-Team, Michael Eiler

| | |
|-|-|
| Organization/Community | team@yavdr.org |
| Developer | eiler.mike@gmail.com or aelo@yavdr.org |
| Website | www.yavdr.org |
| Source | www.github.com/yavdr/vdr-plugin-restfulapi |

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

**Examples**

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

**XML response**

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
**JSON response**

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

**XML response**
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

**JSON response**

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

## Events / EPG

This service returns the epg information.

Since Version 0.2.0 the API returns additional media if a scraper plugin is available.
The data is available within additional_media node and contains additional information about the event such as actors etc.

Images can be retrieved using ScraperImages service.

| Method | URL |
|-|-|
| GET | ```http://<ip>:<port>/events.<format>?chevents=1 ``` |
| GET | ```http://<ip>:<port>/events/<channelid>.<format>?chevents=1&from=<from> ``` |
| GET | ```http://<ip>:<port>/events/<channelid>.<format>?timespan=<timespan>``` |
| GET | ```http://<ip>:<port>/events/<channelid>/<eventid>.<format> ``` |
| GET | ```http://<ip>:<port>/events/<channelid>.<format>?timespan=<timespan>&from=<from>&start=<start>&limit=<limit>``` |

| Param | Description |
|-|-|
| format | The requested format: json, xml or html. |
| channelid | (optional) id of the channel, if not set the plugin will return the whole epg sorted by your channel | list
| timespan | the timespan from which you want to know the event-details, if set to 0 - all events in the future | will be returned
| from | (optional) time in the future when the requested events should end/start (default-value: now) |
| eventid | (optional) if you only want the details of a specific event |
| chevents | (optional) the count of events for each channel |
| chfrom | (optional) start number of the channel, from where events will be returned |
| chto | (optional) end number of the channel, through events will be returned |
| only_count | (optional) if you just want to know the amount of available epg items |
| format | The requested format: json, xml or html. |

**XML response**
```xml
GET http://<ip>:<port>/events/C-1-1079-11110.xml?chevents=1&from=1432577700

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<events xmlns="http://www.domain.org/restfulapi/2011/events-xml">
    <event>
        <param name="id">588333</param>
        <param name="title">Title</param>
        <param name="short_text">Humor (D 2010)</param>
        <param name="description">Description Text</param>
        <param name="channel">C-1-1079-11110</param>
        <param name="channel_name">ZDF HD</param>
        <param name="start_time">1432577700</param>
        <param name="duration">5400</param>
        <param name="table_id">80</param>
        <param name="version">25</param>
        <param name="parental_rating">0</param>
        <param name="vps">1432577700</param>
        <param name="details"></param>
        <param name="images">3</param>
        <param name="components">
            <component stream="2" type="3" language="deu" description="Stereo" />
            <component stream="2" type="3" language="mis" description="Audiodeskription" />
            <component stream="2" type="3" language="mul" description="ohne Originalton" />
            <component stream="3" type="32" language="deu" description="DVB-Untertitel" />
            <component stream="4" type="66" language="deu" description="Dolby Digital 2.0" />
            <component stream="5" type="11" language="deu" description="HDTV" />
        </param>
        <param name="contents"></param>
        <param name="raw_contents"></param>
        <param name="timer_exists">false</param>
        <param name="timer_active">false</param>
        <param name="timer_id"></param>
        <param name="additional_media" type="movie">
            <movie_id>329933</movie_id>
            <title>Title</title>
            <original_title>Original Title</original_title>
            <overview>Overview Text</overview>
            <adult>false</adult>
            <genres>Komödie , Lovestory</genres>
            <release_date>2010-06-01</release_date>
            <runtime>90</runtime>
            <poster path="movies/329933/poster.jpg" width="500" height="750" />
            <fanart path="movies/329933/fanart.jpg" width="1280" height="720" />
            <actor name="Name" role="Role" thumb="movies/actors/actor_49104.jpg"/>
        </param>
    </event>
    <count>1</count>
    <total>257</total>
</events>
```

**JSON response**

```json
GET http://<ip>:<port>/events/C-1-1079-11110.json?chevents=1&from=1432577700

{
    "events": [
        {
            "id": 588333,
            "title": "Title",
            "short_text": "Short Title",
            "description": "Description Text",
            "start_time": 1432577700,
            "channel": "C-1-1079-11110",
            "channel_name": "ZDF HD",
            "duration": 5400,
            "table_id": 80,
            "version": 25,
            "images": 3,
            "timer_exists": false,
            "timer_active": false,
            "timer_id": "",
            "parental_rating": 0,
            "vps": 1432577700,
            "components": [
                {
                    "stream": 2,
                    "type": 3,
                    "language":
                    "deu",
                    "description": "Stereo"
                }, {
                    "stream": 2,
                    "type": 3,
                    "language": "mis",
                    "description": "Audiodeskription"
                }, {
                    "stream": 2, "type": 3,
                    "language": "mul",
                    "description": "ohne Originalton"
                }, {
                    "stream": 3,
                    "type": 32,
                    "language": "deu",
                    "description": "DVB-Untertitel"
                }, {
                    "stream": 4,
                    "type": 66,
                    "language": "deu",
                    "description": "Dolby Digital 2.0"
                }, {
                    "stream": 5,
                    "type": 11,
                    "language": "deu",
                    "description": "HDTV"
                }
            ],
            "contents": [],
            "raw_contents": [],
            "details": [],
            "additional_media": {
                "type": "movie",
                "movie_id": 329933,
                "title": "Title",
                "original_title": "Original Title",
                "tagline": "",
                "overview": "Overview Text",
                "adult": false,
                "collection_name": "",
                "budget": 0,
                "revenue": 0,
                "genres": "Komödie , Lovestory",
                "homepage": "",
                "release_date": "2010-06-01",
                "runtime": 90,
                "popularity": 0,
                "vote_average": 0,
                "poster": "movies/329933/poster.jpg",
                "fanart": "movies/329933/fanart.jpg",
                "collection_poster": "",
                "collection_fanart": "",
                "actors": [
                    {
                        "name": "Name",
                        "role": "Role",
                        "thumb": "movies/actors/actor_49104.jpg"
                    }
                ]
            }
        }
    ],
    "count": 1,
    "total": 257
}
```

### Images

This service returns the requested epg image.


| Method | URL |
|-|-|
| GET | ```http://<ip>:<port>/events/image/<eventid>/<imagenumber>``` |

| Param | Description |
|-|-|
| eventid | Id of the event |
| imagenumber | the number of the image, in the above mentioned examples it would be 0,1 or 2 |

### Search

This service allows you to search for events.

Since Version 0.2.4.0 it is possible to perform a search similar to Searchtimers.

| Method | URL |
|-|-|
| POST | ```http://<ip>:<port>/events/search.<format>?limit=5&start=0``` |

| Param | Description |
|-|-|
| limit | (optional) number of searchresults |
| start | (required if limit is set) start at result |
| date_limit | (optional) UNIX timestamp until events are returned (since 0.2.4.1) |

**HTTP-Body parameters**

| Name | Type | Required | Value Range / Example |
| ---- | ---- | -------- | --------------------- |
| search                | string    | yes |  |
| mode                  | int       | yes | 0=phrase, 1=all words, 2=at least one word, 3=match exactly, 4=regex, 5=fuzzy |
| tolerance             | int       | if mode == 5 | |
| match_case            | boolean   | 	
| use_title             | boolean   | at least one of use_title, use_subtitle, use_description | |
| use_subtitle          | boolean   | at least one of use_title, use_subtitle, use_description | |
| use_description       | boolean   | at least one of use_title, use_subtitle, use_description | |
| content_descriptors   | string    | no | concatted string of content descriptor ids |
| use_ext_epg_info      | boolean   | no |  |
| ext_epg_info          | array     |    | [1#2,2#test] |
| use_time              | boolean   | no |  |
| start_time            | int       | no |  |
| stop_time             | int       | no |  |
| use_channel           | boolean   | no | 	0=no, 1=interval, 2=channel group, 3=only FTA |
| channel_min           | string    | no | 	channel id |
| channel_max           | string    | no | 	channel id |
| channels              | string    | if use_channel > 0 | channel group name |
| use_duration          | boolean   | no |  |
| duration_min          | int       | if use_duration == true | |
| duration_max          | int       | if use_duration == true | |
| use_dayofweek         | boolean   | no |  |
| dayofweek             | int       | if use_dayofweek == true | -127 - 6 |
| blacklist_mode        | int       | no | 	0=global, 1=Selection, 2=all, 3=none |
| blacklist_ids         | array     | if blacklist_mode == 1 | array of blacklist ids |

**Example: Simple search**
```
POST http://<ip>:<port>/events/search.<format>?limit=5&start=0;
```

| Param | Description |
|-|-|
| limit | (optional) number of searchresults |
| start | (required if limit is set) start at result |

**HTTP-Body parameters**

| Param | Description |
|-|-|
| query | (required) |
| mode | (required) 0=phrase, 1=and, 2=or, 3=exact, 4=regex, 5=fuzzy |
| channelid | (optional) if id invalid, the plugin will search on every channel |
| use_title | (optional, default) search in the title |
| use_subtitle | (optional) |
| use_description | (optional) |

**Example**
```json
{"query":"Asterix", "mode":0, "channelid":0,"use_title":true}
```

### Fetch content descriptors

```
GET http://<ip>:<port>/events/contentdescriptors.<format>
```

```json
{
    "content_descriptors": [
        {
            "id": "10",
            "name": "Film/Drama",
            "is_group": true
        },
        {
            "id": "11",
            "name": "Detektiv/Thriller",
            "is_group": false
        }, {
            "id": "12",
            "name": "Abenteuer/Western/Krieg",
            "is_group": false
        },
    ],
    "count": 79,
    "total": 79
}
```

## Info
