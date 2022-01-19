# CONSULTAS_MUMA
Consultas MUMA Geomática

Consultas MUMA

----1. Verificar que no exista viviendas en edificacion---------------------------------------------------------------------------------
select * from revision.viviendas
where usm ilike '2300110186'
order by revision.viviendas.unidad

select * from revision.edificacion
where usm ilike '2300110186'
order by revision.edificacion.edificacion

--- 2.(SI NO HAY VIVIENDA EJECUTARLA SI ESTAN HACER PASO 3)  CARGAR VIVIENDAS DE una o varias USMs en particular y generarles su centroide y su codigo_dane_a-----------------------------
INSERT INTO revision.viviendas(
  unidad, direccion, uso, observaciones, edificacion_id, tiempo, usm, llave)
  SELECT unidad, direccion, uso, observaciones, edificacion, tiempo, usm, llave FROM public.viviendas 
    WHERE TRUE
      AND usm IN ('1951720011')

-- 3. (HAY QUE VALIDAR PRIMERO QUE LAS EDIFICACIONES DE LA USM YA ESTEN EN REVISION) geom centroide
UPDATE revision.viviendas SET geom = e.centroide FROM revision.edificacion e
  WHERE revision.viviendas.usm = '1951720011'  
    AND revision.viviendas.usm = e.usm AND revision.viviendas.edificacion_id = e."_id";
  
-- 4. cod_dane_a
UPDATE revision.viviendas SET cod_dane_a = e.mgn_manzana FROM revision.edificacion e
  WHERE revision.viviendas.usm = '1951720011'
    AND revision.viviendas.usm = e.usm AND revision.viviendas.edificacion_id = e."_id";
  
-- 5. Crear GEOMETRIA de VIVIENDAS
UPDATE revision.viviendas SET geom = ST_Centroid(e.geometry) FROM revision.edificacion e
  WHERE revision.viviendas.usm = '1951720011'  
    AND revision.viviendas.usm = e.usm AND revision.viviendas.edificacion_id = e."_id"
  
-- 6. AJUSTAR ETIQUETAS  (APLICA PARA CUANDO SE AJUSTE EDIFICACIONES)
  UPDATE   revision.edificacion SET vivienda_ini = v.min, vivienda_fin = v.max FROM (
    SELECT usm, edificacion_id, MIN(unidad), MAX(unidad) FROM revision.viviendas 
      GROUP BY usm, edificacion_id
      ORDER BY usm, edificacion_id
  ) v
  WHERE TRUE AND v.usm = '5000610032'
    AND revision.edificacion.usm = v.usm
    AND revision.edificacion._id = v.edificacion_id;
    
  UPDATE revision.edificacion SET viviendas = vivienda_fin+1 - vivienda_ini WHERE usm = '5000610032'

-- 7. OK EN CONSECUTIVOS DE VIVIENDAS-------------------------------------------------------------------------------------------------------
  UPDATE revision.viviendas SET ok_orden = 1 WHERE usm = '1951720011'

