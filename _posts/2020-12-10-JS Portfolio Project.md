---
layout: post
title: JS Portfolio Project Flatiron School (mod4 project)!
---

JS project

For my project I took the same subject as with the Sinatra project, as a hobby I have my radio amateur license. Especially for long distances we like to keep a logbook, maybe a bit to show off.

Because of this the framework that I needed was not an issue and was clear in my head.

Until now I kept my logs in two different places. The Logbook of the World (LotW) hosted by the ARRL (American Radio Relay League) and another online logbook hosted by eQSL.cc. At both sites I was able to download my contacts in ADI format, a format that is used by many logging programs that would make importing the data fairly easy.

Already while working on my Sinatra I came across a ruby script adif_to_sql.rb. it is a command line script and need some adjustments to get it working.
At least it got me from 
{% highlight ruby %}
<eoh>

<APP_LoTW_OWNCALL:4>K5GT
<STATION_CALLSIGN:4>K5GT
<CALL:6>CT7ACG
<BAND:3>20M
<FREQ:8>14.07510
<MODE:3>FT8
<APP_LoTW_MODEGROUP:4>DATA
<QSO_DATE:8>20190801
<APP_LoTW_RXQSO:19>2019-08-01 22:16:03 // QSO record inserted/modified at LoTW
<TIME_ON:6>221400
<APP_LoTW_QSO_TIMESTAMP:20>2019-08-01T22:14:00Z // QSO Date & Time; ISO-8601
<QSL_RCVD:1>Y
<QSLRDATE:8>20200921
<APP_LoTW_RXQSL:19>2020-09-21 11:47:03 // QSL record matched/modified at LoTW
<DXCC:3>272
<COUNTRY:8>PORTUGAL
<APP_LoTW_DXCC_ENTITY_STATUS:7>Current
<PFX:3>CT7
<APP_LoTW_2xQSL:1>Y
<GRIDSQUARE:6>IM57VF
<CQZ:2>14
<APP_LoTW_CQZ_Inferred:1>Y // from DXCC entity
<ITUZ:2>37
<APP_LoTW_ITUZ_Inferred:1>Y // from DXCC entity
<eor>
{% endhighlight %}

to

{% highlight ruby %}
insert into contacts ('app_lotw_owncall','station_callsign','call','band','freq','mode','app_lotw_modegroup','qso_date','app_lotw_rxqso','time_on','app_lotw_qso_timestamp','qsl_rcvd','qslrdate','app_lotw_rxqsl','dxcc','country','app_lotw_dxcc_entity_status','pfx','app_lotw_2xqsl','gridsquare','cqz','app_lotw_cqz_inferred','ituz','app_lotw_ituz_inferred') values ('K5GT
','K5GT
','CT7ACG
','20M
','14.07510
','FT8
','DATA
','20190801
','2019-08-01 22:16:03 // QSO record inserted/modified at LoTW
','221400
','2019-08-01T22:14:00Z // QSO Date & Time; ISO-8601
','Y
','20200921
','2020-09-21 11:47:03 // QSL record matched/modified at LoTW
','272
','PORTUGAL
','Current
','CT7
','Y
','IM57VF
','14
','Y // from DXCC entity
','37
','Y // from DXCC entity
');
{% endhighlight %}
