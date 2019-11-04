# import polygone to postgis using openlayers and geoserver

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).

### Configure PostgreSQL

```
-- DROP INDEX public.wfs_polygon_idx;
-- DROP TABLE public.wfs_polygon;
-- DROP SEQUENCE public.wfs_polygon_id_seq;
CREATE SEQUENCE public.wfs_polygon_id_seq;
ALTER SEQUENCE public.wfs_polygon_id_seq
    OWNER TO postgres;

CREATE TABLE public.wfs_polygon
(
  	id integer NOT NULL DEFAULT nextval('wfs_polygon_id_seq'::regclass),
  	geometry geometry(Polygon,3857),
    CONSTRAINT wfs_polygon_pkey PRIMARY KEY (id)
)
WITH (
    OIDS=FALSE
)
TABLESPACE pg_default;

ALTER TABLE wfs_polygon
    OWNER TO postgres;

CREATE INDEX wfs_polygon_idx
    ON wfs_polygon
    USING gist
    (geometry)
    TABLESPACE pg_default;
```
