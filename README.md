# rd_gebaeude_ausserhalb_bauzone

```

java -jar /Users/stefan/apps/ili2h2gis-4.5.0/ili2h2gis-4.5.0.jar --dbfile sogis --disableValidation --defaultSrsCode 2056 --strokeArcs --nameByTopic --models SO_AGI_MOpublic_20190424 --dbschema agi_mopublic_pub --createGeomIdx --doSchemaImport --import /Users/stefan/Downloads/ch.so.agi.mopublic_xtf/ch.so.agi.mopublic.xtf

java -jar /Users/stefan/apps/ili2h2gis-4.5.0/ili2h2gis-4.5.0.jar --dbfile sogis --disableValidation --defaultSrsCode 2056 --strokeArcs --nameByTopic --models SO_Nutzungsplanung_Publikation_20190909 --dbschema arp_nutzungsplanung_pub --createGeomIdx --doSchemaImport --import /Users/stefan/Downloads/ch.so.arp.nutzungsplanung_xtf/ch.so.arp.nutzungsplanung.xtf
```


```
WITH gebaeude AS
(
    SELECT 
        --count(*)
        t_id AS g_tid, bfs_nr, ST_PointOnSurface(geometrie) AS point, geometrie
    FROM
        AGI_MOPUBLIC_PUB.MOPUBLIC_BODENBEDECKUNG 
    WHERE 
        ART_TXT = 'Gebaeude'
) 
,
bauzone AS 
( 
    SELECT 
        t_id AS b_tid, geometrie, typ_kt
    FROM
        ARP_NPL_PUB.NUTZUNGSPLANUNG_GRUNDNUTZUNG 
    WHERE 
        CAST(SUBSTRING(typ_kt from 2 for 2) AS INTEGER) < 20
        OR
        CAST(SUBSTRING(typ_kt from 2 for 2) AS INTEGER) = 43
)
,
intersects AS 
(
    SELECT
        gebaeude.g_tid,
		gebaeude.bfs_nr,
        gebaeude.geometrie,
        bauzone.b_tid
        --count(*)
    FROM
        gebaeude 
        LEFT JOIN bauzone 
        ON bauzone.geometrie && gebaeude.geometrie
        AND
        ST_Intersects(gebaeude.point, (bauzone.geometrie))
)
SELECT 
    *
FROM 
    intersects
where
	b_tid IS NULL
```