# Prompt: convertir una imagen de mapa con símbolos militares en un GeoJSON para el Sistema de Anotación Cartográfica CCON-D

Copie todo lo que está debajo de la línea y péguelo en el asistente de IA (con capacidad de ver imágenes), adjuntando la imagen del mapa.

---

## ROL Y OBJETIVO

Usted es un analista cartográfico militar. Recibirá una **imagen de un mapa con símbolos y gráficos militares**: unidades, puestos de mando, vehículos, equipos, armas, instalaciones, medidas de coordinación y control (MCC), flechas, tareas tácticas, límites y líneas. Su tarea es **transcribirla a un único archivo GeoJSON** (`FeatureCollection`) que pueda cargarse con el botón **«⇪ Importar GeoJSON»** de la plataforma CCON-D, de modo que cada símbolo aparezca en su posición geográfica y con el símbolo más fiel posible al de la imagen.

La plataforma **no lee imágenes de símbolos**: dibuja cada símbolo a partir de parámetros (afiliación, unidad base, tipo, escalón, designación, modificadores, condición). Por eso debe **traducir cada símbolo visto en la imagen a esos parámetros**, usando únicamente los valores permitidos que se listan abajo.

## REGLAS GENERALES

1. Responda con **un solo bloque de código JSON válido** (sin comentarios, sin comas finales, comillas dobles). Después del bloque, añada una tabla breve de **dudas y supuestos** (ver sección 7). No escriba nada más.
2. **Coordenadas en WGS84, orden `[longitud, latitud]`** (no al revés), con 5–6 decimales. En Colombia la longitud es negativa (≈ −67 a −79) y la latitud positiva (≈ −4 a 13).
3. **Un símbolo = un `Feature` de tipo `Point`.** No agrupe varios símbolos en un mismo punto.
4. **No invente.** Si un dato no se ve o no se puede leer, use el valor por defecto (indicado abajo) y anótelo en la tabla de dudas. Nunca invente designaciones, coordenadas ni escalones.
5. Use **solo los valores exactos** de las listas permitidas (en minúsculas, tal como están escritos). Un valor fuera de lista hace que el símbolo se dibuje como genérico.

## 1. GEORREFERENCIACIÓN (ubicar cada símbolo)

- Si la imagen trae **coordenadas, cuadrícula, MGRS/UTM, graticula de latitud/longitud, escala y norte**, úselas para calcular la posición de cada símbolo.
- Si no trae coordenadas, identifique **al menos 3–4 puntos de control** reconocibles (cabeceras municipales, cruces de ríos y vías, costas, límites departamentales, aeropuertos, etc.), asígneles sus coordenadas reales conocidas y calcule la posición de cada símbolo por interpolación. Indique en la tabla de supuestos qué puntos de control usó.
- Si la imagen **no permite georreferenciar con confianza**, no adivine: devuelva igualmente el GeoJSON con la mejor estimación y márquelo claramente como «posición aproximada» en la tabla de dudas.
- La **punta/centro del símbolo** es la posición. En símbolos con una línea de anclaje o flecha hacia el punto, tome el punto señalado.

## 2. ESTRUCTURA DE CADA SÍMBOLO

```json
{
  "type": "Feature",
  "properties": {
    "nombre": "BR 9 - BIM 4",
    "supply": 100,
    "mil": {
      "affiliation": "friendly",
      "unitBase": "infanteria",
      "unitType": "infantry",
      "echelon": "battalion",
      "designation": "BIM 4",
      "modifiers": [],
      "condition": "present"
    }
  },
  "geometry": { "type": "Point", "coordinates": [-76.5321, 3.4516] }
}
```

- `nombre`: texto con el que se identifica el símbolo en la lista de capas (use la designación y/o el nombre de la unidad tal como figura en la imagen; si no hay, descríbalo, p. ej. «Puesto de mando sin identificar»).
- `supply`: déjelo en `100`.
- `mil`: parámetros del símbolo (secciones 3 y 4). **Siempre incluya los siete campos.**

## 3. VALORES PERMITIDOS

**`affiliation`** (por la forma y color del marco):
- `friendly` — amigo (marco azul, rectángulo)
- `hostile` — enemigo (marco rojo, rombo)
- `neutral` — neutral (marco verde, cuadrado)
- `unknown` — desconocido (marco amarillo, trébol/«flor»)
Si el marco no permite distinguirlo, use el color de la imagen como guía; si tampoco, `unknown`.

**`echelon`** (por las marcas sobre el marco; vacío `""` si no hay):
`team` (Ø), `squad` (•), `section` (••), `platoon` (•••), `company` (I), `battalion` (II), `regiment` (III; también «Grupo»), `brigade` (X), `division` (XX), `corps` (XXX), `army` (XXXX).

**`condition`**: `present` (por defecto, línea continua) · `planned` (línea discontinua / futuro) · `damaged` (dañado) · `destroyed` (destruido).

**`modifiers`** (lista; vacía `[]` si no hay): `staff` (puesto de comando), `dummy` (simulado), `installation` (instalación), `taskforce` (fuerza de tareas), `direction` (dirección).

**`unitBase`** y **`unitType`**: `unitType` debe pertenecer a su `unitBase`, según esta tabla (la columna derecha son los `unitType` válidos):

| `unitBase` | `unitType` válidos |
|---|---|
| `generic` | `generic` (genérica), `illicit_economy` (economías ilícitas $) |
| `infanteria` | `infantry` (regular), `mechanized`, `bafur`, `biter`, `biacm`, `ev` (BAEV) |
| `caballeria` | `cavalryligera` (ligera), `cavalry` (blindada), `gmar` |
| `artilleria` | `artillery` (campaña), `airdefense` (antiaérea), `airdefensemissile`, `mortar`, `obus`, `badre`, `badra` |
| `ingenieros` | `engineer` |
| `comunicaciones` | `signal`, `gtm`, `commandpost` (puesto de mando), `accionintegral` |
| `inteligencia` | `radar`, `radarmilitar`, `evipe`, `imi` |
| `fuerzasespeciales` | `specialforces`, `sof`, `ggcar`, `ggmag`, `ggsan`, `ggmmag`, `futam`, `circo` |
| `aviacion` | `aviation` (ala fija), `rotarywing` (ala rotatoria), `baseaerea` |
| `logistica` | `logistics`, `aspc`, `ammodepot`, `udiv`, `abastecimiento`, `intendencia`, `brlog`, `claseI` … `claseX` (clases de abastecimiento: `claseI`, `claseII`, `claseIII`, `claseIV`, `claseV`, `claseVI`, `claseVII`, `claseVIII`, `claseIX`, `claseX`), `lavanderia`, `agua` |
| `sanidad` | `medical` |
| `mantenimiento` | `mantenimiento` |
| `transporte` | `transporte` (terrestre), `buque`, `navalelement` |
| `policiamilitar` | `policiamilitar`, `bapom` |
| `accion_integral` | `baid`, `cpaid` |

**Subtipos de movilidad** (válidos para todas las `unitBase` excepto `generic`): `armor` (blindada), `aerotransportada`, `amphibious` (anfibia), `motorized` (motorizada), `wheeled` (a rueda), `unknown_unit` (desconocido ?). Se dibujan **superpuestos** al símbolo propio de la unidad base. En `caballeria` no use `armor`: la blindada es `cavalry`.

## 4. CÓMO TRADUCIR EL DIBUJO A PARÁMETROS

1. **Marco** → `affiliation`.
2. **Marcas sobre el marco** → `echelon`.
3. **Pictograma dentro del marco** → `unitBase` + `unitType`:
   - X cruzada = infantería (`infanteria`/`infantry`); X con óvalo = infantería mecanizada o blindada según el caso; diagonal sola = caballería; óvalo (cadena) = blindada; círculo relleno = artillería; castillo = ingenieros; zigzag/rayo = comunicaciones; ala/hélice = aviación; cruz = sanidad; etc.
   - Si el pictograma combina uno propio de arma con un óvalo (blindado), una rueda, un paracaídas, etc., elija la `unitBase` del arma y el subtipo de movilidad correspondiente (`armor`, `wheeled`, `aerotransportada`…) como `unitType`.
   - Si el símbolo lleva **siglas** (BAFUR, BITER, GGCAR, FUTAM, BADRA, BAPOM, etc.), use el `unitType` con ese nombre.
   - Si no reconoce el pictograma: `unitBase: "generic"`, `unitType: "generic"` y anótelo en la tabla de dudas.
4. **Texto junto al símbolo** (designación, nombre de la unidad, número) → `designation` (solo el identificador corto, p. ej. «91-9», «BIM 4», «BR 9») y/o `nombre` (más descriptivo). No ponga allí fechas, coordenadas ni comentarios.
5. **Marcas adicionales** (banderín = puesto de comando → `staff`; rombo/guion de instalación → `installation`; línea discontinua → `condition: "planned"`; tachado → `destroyed`; fuerza de tareas → `taskforce`).

## 5. OTROS ELEMENTOS DEL MAPA (no son símbolos de unidad)

Las secciones 2 a 4 describen los **símbolos de unidad**. Para el resto, siga este orden de decisión:

1. **Vehículos, equipos, armas, infraestructura, blancos, MCC terrestres (puntos, líneas y zona), líneas defensivas y límites con escalón**: están especificados en el **ANEXO A** con sus valores exactos. Úselos tal cual.
2. **Flechas, tareas tácticas, MCC aéreas / de apoyo de fuego / logísticas, ingenieros, línea ofensiva y similares**: son demasiado complejos para describirlos de memoria. Requieren la **plantilla exportada de la plataforma** (**ANEXO B**). Si no hay plantilla para ese tipo, no lo invente: déjelo fuera y anótelo como «requiere plantilla».
3. **Líneas, áreas y textos simples** (sin significado militar especial):
   - **Línea**: `LineString` con `properties`: `{"nombre":"...", "color":"#1F3864", "weight":3, "opacity":1, "dashArray":""}` (`dashArray` vacío = continua; `"8 6"` = discontinua).
   - **Área**: `Polygon` (anillo cerrado: el primer y el último punto son iguales) con `{"nombre":"...", "color":"#B8962E", "weight":2, "fillColor":"#B8962E", "fillOpacity":0.2}`.
   - **Texto suelto**: `Point` con `{"nombre":"...", "text":{"text":"Texto","fontSize":14,"color":"#000000","bgColor":"#ffffff","bgOpacity":0,"hasBorder":false,"borderColor":"#000000","rotation":0,"boxWidth":150,"boxHeight":50}}`.

Todos los elementos se entregan en el **mismo** `FeatureCollection`. Si el usuario solo pidió ciertos elementos, incluya únicamente esos.

## 6. VERIFICACIÓN ANTES DE ENTREGAR (haga esta revisión usted mismo)

- [ ] El JSON es válido y es un `FeatureCollection` con `features`.
- [ ] Hay **un `Feature` por cada símbolo o gráfico** de la imagen (cuéntelos: imagen = archivo), salvo los marcados «requiere plantilla».
- [ ] Cada punto tiene `[longitud, latitud]` (longitud primero) y cae dentro de la zona que muestra el mapa.
- [ ] Todos los `mil` tienen `affiliation`, `unitBase`, `unitType`, `echelon`, `designation`, `modifiers`, `condition`, y cada valor está en la lista permitida; el `unitType` pertenece a su `unitBase`.
- [ ] Ninguna designación fue inventada y ningún elemento complejo se armó sin plantilla.

## 7. FORMATO DE LA RESPUESTA

1. El bloque de código con el GeoJSON completo (nómbrelo mentalmente `simbolos_<tema>.geojson`).
2. Una tabla «**Dudas y supuestos**» con columnas: *Símbolo (nombre)* · *Qué no quedó claro* · *Valor que usó*. Incluya los puntos de control de georreferenciación, los símbolos con posición aproximada y los elementos que quedaron fuera por «requiere plantilla».

## CÓMO CARGAR Y AJUSTAR (para quien recibe el resultado)

1. Guarde el bloque JSON en un archivo de texto con extensión `.geojson`.
2. En la plataforma, pestaña **Edición**, use **«⇪ Importar GeoJSON»**: se crea una capa nueva con el nombre del archivo.
3. Compare el resultado con la imagen original. Para corregir un símbolo, selecciónelo en el mapa y edite sus parámetros en el panel de propiedades (afiliación, unidad base, tipo, escalón, designación, modificadores, condición) o arrástrelo a su posición exacta. Lo que quedó como «requiere plantilla» se dibuja a mano con la barra de herramientas.


## ANEXO A · OTROS ELEMENTOS DE LA PLATAFORMA (además de las unidades)

La plataforma dibuja más cosas que símbolos de unidades. Cada elemento tiene su propia estructura en `properties` (el `geometry` sigue las reglas de siempre). Los elementos de esta sección **fueron probados**: al importarlos, la plataforma los reconoce y los dibuja. Todos aceptan además `"nombre"`.

### A.1 Vehículos → `Point` con `properties.vehiculo`

```json
{"type":"Feature","properties":{"nombre":"Tanque 1","vehiculo":{"affiliation":"friendly","condition":"present","tipoVehiculo":"terrestre","category":"combate","tipo":"tanque","leftLabel":""}},"geometry":{"type":"Point","coordinates":[-76.5,3.4]}}
```

- `affiliation`: igual que en unidades (`friendly`, `hostile`, `neutral`, `unknown`).
- `condition`: `present` (Presente) · `planned` (Futuro) · `dummy` (Simulado).
- `tipoVehiculo` y `category` (la categoría debe corresponder al tipo de vehículo):
  - `terrestre`: `combate`, `ingenieros`, `logisticos`
  - `aereo`: `ala_fija`, `ala_rotatoria`, `uav`
  - `naval`: `superficie`, `submarinos`
- `leftLabel`: texto corto a la izquierda del símbolo (matrícula, designación); `""` si no hay.
- `tipo` según la categoría (use exactamente estas claves):
  - **terrestre / combate**: `uso_general` (Vehículo de uso general) · `rueda` (Vehículo a rueda) · `todo_terreno_blindado` (Vehículo todo terreno con protección blindada) · `tanque` (Tanque) · `tanque_liviano` (Tanque liviano) · `tanque_mediano` (Tanque mediano) · `tanque_pesado` (Tanque pesado) · `tanque_rueda` (Tanque a rueda) · `tanque_mediano_rueda` (Tanque mediano a rueda) · `tanque_pesado_rueda` (Tanque pesado a rueda) · `vctp` (Vehículo de combate de transporte de personal) · `vctp_rueda` (Vehículo de combate de transporte de personal a rueda) · `vci` (Vehículo de combate de infantería) · `vci_rueda` (Vehículo de combate de infantería a rueda) · `vcrt` (Vehículo de combate recuperador de tanque) · `vcrt_liviano` (Vehículo de combate recuperador de tanque liviano) · `vcrt_mediano` (Vehículo de combate recuperador de tanque mediano) · `vcrt_pesado` (Vehículo de combate recuperador de tanque pesado) · `vctp_recuperador` (VCTP recuperador)
  - **terrestre / ingenieros**: `ing_puente_flotante` (Puente flotante) · `ing_rueda` (Vehículo a rueda de ingenieros) · `ing_transporte_puente` (Vehículo transporte de puente) · `ing_lanzador_puente` (Vehículo lanzador de puente) · `ing_movimiento_tierra` (Vehículo de movimiento de tierra) · `ing_sembrador_minas` (Vehículo sembrador de minas) · `ing_minado_dinamico` (Vehículo de minado dinámico) · `ing_barreminas` (Vehículo con barreminas) · `ing_brecha_explosiva` (Vehículo blindado de apertura de brecha con carga explosiva) · `ing_brecha_arado` (Vehículo de apertura de brecha con arado/dozer)
  - **terrestre / logisticos**: `log_ambulancia` (Ambulancia) · `log_ambulancia_blindada` (Ambulancia blindada) · `log_locomotora` (Locomotora) · `log_vagon` (Vagón de tren) · `log_rueda_sanidad` (Vehículo a rueda de sanidad) · `log_vehiculo_rueda` (Vehículo a rueda) · `log_cisterna_agua` (Vehículo cisterna de agua) · `log_cl3` (Vehículo de transporte de efectos Cl III)
  - **aereo / ala_fija**: `af_generico` (Vehículo aéreo) · `af_militar` (Vehículo aéreo militar) · `af_fijo` (Vehículo aéreo de ala fija) · `af_bombardero` (Bombardero) · `af_caza` (Caza) · `af_interceptor` (Interceptor) · `af_entrenamiento` (Entrenamiento) · `af_ataque` (Ataque) · `af_vstol` (VSTOL) · `af_tanker` (Tanker) · `af_carga` (Carga) · `af_cme` (Contramedidas Electrónicas) · `af_medevac` (MEDEVAC) · `af_reconocimiento` (Reconocimiento) · `af_alerta_temprana` (Alerta temprana) · `af_reconocimiento_electronico` (Reconocimiento electrónico) · `af_reconocimiento_fotografico` (Reconocimiento fotográfico) · `af_patrulla` (Patrulla) · `af_asuw` (ASUW) · `af_contraminas` (Contraminas) · `af_utilitario` (Utilitario) · `af_c3i` (C3I) · `af_csar` (CSAR) · `af_puesto_comando` (Puesto Comando) · `af_sof` (SOF)
  - **aereo / ala_rotatoria**: `ar_generico` (Vehículo de ala rotatoria) · `ar_ataque` (De ataque) · `ar_asw` (De guerra antisubmarina) · `ar_utilitario` (Utilitario) · `ar_csar` (CSAR) · `ar_reconocimiento` (De Reconocimiento) · `ar_carga` (De Carga) · `ar_medevac` (MEDEVAC) · `ar_entrenamiento` (De Entrenamiento) · `ar_sof` (De SOF) · `ar_puesto_comando` (De Puesto Comando)
  - **aereo / uav**: `uav_ala_fija` (Dron de Ala Fija) · `uav_ala_rotatoria` (Dron de Ala Rotatoria)
  - **naval / superficie**: `sup_nave_combate` (Nave de combate) · `sup_nave_linea` (Nave de línea) · `sup_portaaviones` (Portaaviones) · `sup_crucero_batalla` (Crucero de batalla) · `sup_crucero` (Crucero) · `sup_destructor` (Destructor) · `sup_fragata` (Fragata) · `sup_combate_anfibio` (Nave de combate anfibio) · `sup_asalto_anfibio` (Nave de asalto anfibio) · `sup_desembarco_anfibio` (Nave de desembarco anfibio) · `sup_lancha_desembarco` (Lancha de desembarco anfibio) · `sup_barreminas` (Nave barreminas) · `sup_minadora` (Nave minadora) · `sup_barreminas_ms` (Nave barreminas (MS)) · `sup_cazaminas` (Nave cazaminas) · `sup_contramedidas_antiminas` (Nave de contramedidas antiminas) · `sup_dron_contramedidas` (Dron de contramedidas antiminas) · `sup_patrullera_combate` (Patrullera de combate) · `sup_patrullera_antisubmarina` (Patrullera de combate antisubmarino) · `sup_patrullera_antisuperficie` (Patrullera de combate antisuperficie) · `sup_hovercraft` (Hovercraft) · `sup_estacion_combate` (Estación de combate) · `sup_estacion_vigilancia` (Estación de vigilancia) · `sup_estacion_antisubmarina` (Estación antisubmarina) · `sup_grupo_combate` (Grupo de combate) · `sup_fuerza_tarea` (Fuerza de tarea) · `sup_grupo_tareas` (Grupo de tareas) · `sup_unidad_tareas` (Unidad de tareas) · `sup_grupo_convoy` (Grupo de convoy) · `sup_no_combatiente` (Nave no combatiente) · `sup_reabastecimiento` (Nave de reabastecimiento) · `sup_soporte` (Nave de soporte) · `sup_inteligencia` (Nave de inteligencia) · `sup_servicio_apoyo` (Nave de servicio y apoyo) · `sup_buque_hospital` (Buque hospital) · `sup_hovercraft_nc` (Hovercraft no combatiente) · `sup_estacion_nc` (Estación no combatiente) · `sup_estacion_rescate` (Estación de rescate) · `sup_mercante` (Nave mercante) · `sup_mercante_carga` (Nave mercante de carga) · `sup_mercante_roro` (Nave mercante RO/RO) · `sup_mercante_tanque` (Nave mercante tanque) · `sup_mercante_remolque` (Nave mercante remolque) · `sup_mercante_ferry` (Nave mercante ferry) · `sup_mercante_pasajeros` (Nave mercante de pasajeros) · `sup_mercante_hazmat` (Nave mercante de materiales peligrosos) · `sup_mercante_remolcadora` (Nave mercante remolcadora) · `sup_pesquero` (Pesquero) · `sup_pesquero_deriva` (Pesquero de deriva) · `sup_draga` (Draga) · `sup_pesquero_arrastre` (Pesquero de arrastre) · `sup_placer` (Nave de placer) · `sup_policial` (Nave policial)
  - **naval / submarinos**: `sub_submarino` (Submarino) · `sub_nuclear` (Submarino nuclear) · `sub_convencional` (Submarino convencional) · `sub_otros_sumergibles` (Otros sumergibles) · `sub_estacion` (Estación de submarinos) · `sub_estacion_antisub` (Estación de submarinos antisubmarinos) · `sub_arma` (Arma submarina) · `sub_torpedo` (Torpedo) · `sub_mina` (Mina) · `sub_mina_anclada` (Mina (anclada)) · `sub_mina_flotante` (Mina (flotante)) · `sub_senuelo` (Señuelo submarino) · `sub_mina_senuelo` (Mina señuelo)

### A.2 Equipos → `Point` con `properties.equipo`

```json
{"type":"Feature","properties":{"nombre":"Repetidor","equipo":{"affiliation":"friendly","condition":"present","category":"radioelectricos","tipo":"eq_rad_repetidor","leftLabel":""}},"geometry":{"type":"Point","coordinates":[-76.3,3.4]}}
```

`condition` y `affiliation` como en vehículos. `category`: hoy solo está cargada `radioelectricos` (las categorías `ingenieros` y `especiales` están vacías). `tipo`: `eq_rad_portatil_mochilar` (Radio portátil mochilar) · `eq_rad_portatil_vehicular` (Radio portátil vehicular) · `eq_rad_portatil_manual` (Radio portátil manual) · `eq_rad_estacion_fija` (Estación radioeléctrica fija) · `eq_rad_manual_vhf` (Radio portátil manual VHF) · `eq_rad_manual_uhf` (Radio portátil manual UHF) · `eq_rad_repetidor` (Repetidor de radio) · `eq_rad_integracion_radio_radio` (Equipo de integración Radio-Radio) · `eq_rad_integracion_radio_cableado` (Equipo de integración radio-cableado) · `eq_rad_receptor` (Equipo de radio receptor) · `eq_rad_multicanal` (Radio multicanal) · `eq_rad_multicanal_repetidor` (Radio multicanal repetidor) · `eq_rad_radiogoniometro` (Radiogoniómetro) · `eq_rad_engano_imitativo` (Engaño imitativo) · `eq_rad_interferencia` (Equipo de interferencia radioeléctrica) · `eq_rad_ctvm` (Centro de transmisión videomóvil) · `eq_rad_ccmi` (Centro de conmutación móvil integrado) · `eq_rad_ctcm` (Centro troncalizador de comunicaciones móviles) · `eq_rad_telefono_satelital` (Teléfono satelital de campaña) · `eq_rad_terminal_sat_remolcable` (Terminal satelital de campaña remolcable) · `eq_rad_terminal_sat_transportable` (Terminal satelital de campaña transportable) · `eq_rad_terminal_sat_portatil` (Terminal satelital portátil) · `eq_rad_radioenlace_digital` (Radioenlace digital de campaña) · `eq_rad_central_telefonica` (Central telefónica) · `eq_rad_centro_mensajes` (Centro de mensajes) · `eq_rad_telefono_campana` (Teléfono de campaña) · `eq_rad_escucha_alambrica` (Puesto de escucha alámbrica) · `eq_rad_generador` (Generador) · `eq_rad_palomar` (Palomar) · `eq_rad_estafeta` (Estafeta) · `eq_rad_estafeta_rueda` (Estafeta a rueda) · `eq_rad_estafeta_aereo` (Estafeta aéreo).

### A.3 Armas → `Point` con `properties.arma`

```json
{"type":"Feature","properties":{"nombre":"Obús","arma":{"affiliation":"friendly","condition":"present","category":"apoyo_fuego","tipo":"obus_liviano","radioAccionEnabled":true,"radioAccionKm":8,"leftLabel":""}},"geometry":{"type":"Point","coordinates":[-76.2,3.4]}}
```

`category`: `portatiles` o `apoyo_fuego`. `radioAccionEnabled` (true/false) dibuja un círculo de radio de acción de `radioAccionKm` kilómetros (solo si la imagen muestra el alcance).
- **portatiles**: `fusil_generico` (Fusil (Genérico)) · `fusil` (Fusil) · `fusil_semiautomatico` (Fusil Semiautomático (FAL)) · `fusil_automatico` (Fusil Automático (FAP)) · `ametralladora_generico` (Ametralladora (Genérico)) · `ametralladora_liviana` (Ametralladora (Liviana)) · `ametralladora_mediana` (Ametralladora Mediana) · `ametralladora_pesada` (Ametralladora Pesada) · `arma_antitanque` (Arma antitanque) · `arma_antitanque_liviana` (Arma Antitanque Liviana) · `arma_antitanque_mediana` (Arma Antitanque Mediana) · `arma_antitanque_pesada` (Arma Antitanque Pesada) · `arma_antitanque_sin_retroceso` (Arma Antitanque sin Retroceso) · `lanzacohetes` (Lanzacohetes) · `lanzacohetes_liviano` (Lanzacohetes Liviano) · `lanzacohetes_mediano` (Lanzacohetes mediano) · `lanzacohetes_pesado` (Lanzacohetes Pesado) · `lanzagranadas` (Lanzagranadas) · `lanzagranadas_liviano` (Lanzagranadas Liviano) · `lanzagranadas_mediano` (Lanzagranadas Mediano) · `lanzagranadas_pesado` (Lanzagranadas Pesado) · `misil_antitanque` (Misil Antitanque) · `misil_antitanque_liviano` (Misil Antitanque Liviano) · `misil_antitanque_mediano` (Misil Antitanque Mediano) · `misil_antitanque_pesado` (Misil Antitanque Pesado) · `mortero` (Mortero) · `mortero_liviano` (Mortero Liviano) · `mortero_mediano` (Mortero Mediano) · `mortero_pesado` (Mortero Pesado)
- **apoyo_fuego**: `obus` (Obús) · `obus_liviano` (Obús Liviano) · `obus_liviano_autopropulsado` (Obús Liviano Autopropulsado) · `obus_mediano` (Obús Mediano) · `obus_mediano_autopropulsado` (Obús Mediano Autopropulsado) · `obus_pesado` (Obús Pesado) · `obus_pesado_autopropulsado` (Obús Pesado Autopropulsado) · `lanzamisiles_ss` (Lanzamisiles Superficie-Superficie) · `lanzamisiles_ss_corto` (Lanzamisiles S-S Corto Alcance) · `lanzamisiles_ss_mediano` (Lanzamisiles S-S Mediano Alcance) · `lanzamisiles_ss_largo` (Lanzamisiles S-S Largo Alcance) · `lanzamisiles_daa` (Lanzamisiles de Defensa Antiaérea) · `lanzamisiles_saa_corto` (Lanzamisiles S-A Corto Alcance) · `lanzamisiles_saa_mediano` (Lanzamisiles S-A Mediano Alcance) · `lanzamisiles_saa_largo` (Lanzamisiles S-A Largo Alcance) · `canon_daa` (Cañón de Defensa Antiaérea) · `canon_daa_liviano` (Cañón DAA Liviano) · `canon_daa_mediano` (Cañón DAA Mediano) · `canon_daa_pesado` (Cañón DAA Pesado) · `radar` (Radar) · `sensor` (Sensor) · `uas_ala_fija` (Sistema Aéreo No Tripulado de Ala Fija) · `uas_ala_rotativa` (Sistema Aéreo No Tripulado de Ala Rotativa) · `met` (Estación Meteorológica) · `of_enlace` (Oficial de Enlace) · `oea` (Oficial de Enlace de Artillería)

### A.4 Infraestructura → `Point` con `properties.infra`

`tipo`: `minero` (Minero) · `industrial` (Industrial) · `hidrocarburos` (Hidrocarburos) · `portuario` (Portuario) · `energetico` (Energético) · `transporte` (Transporte) · `fronterizo` (Fronterizo) · `hidrico` (Hídrico) · `institucional` (Institucional) · `aeropuerto` (Aeropuertos) · `torre_alta_tension` (Torre de alta tensión) · `hospital` (Hospital) · `emisora` (Emisora).
- Tipos generales (todos menos `hospital` y `emisora`): `{"tipo":"energetico","codigo":"E-1","activo":"Subestación","descripcion":"","criticidad":"A","showCodigo":true}`. `criticidad`: `A` (crítica), `B` (alta), `C` (media).
- `hospital`: `{"tipo":"hospital","nivel":"","denominacion":"","municipio":"","departamento":"","showCodigo":true}`.
- `emisora`: `{"tipo":"emisora","identificacion":"","fuerza":"","unidad":"","herramienta":"","municipio":"","departamento":"","cobertura":"","estado":"","observaciones":"","showCodigo":true}`.

### A.5 Blancos → `Point` con `properties.target`

`{"target":{"number":"AA001","blanco":"PC enemigo","unidad":"","tipo":"","mision":"","fase":"","medio":"","ambito":"","ilaRef":""}}`. `number` es el código visible junto a la cruz.

### A.6 Medidas de Coordinación y Control (MCC) terrestres

- **Puntos** → `Point` con `properties.mcc`: `{"type":"control","text":"PC1","scale":1,"color":"#1F3864","fillOpacity":1}`. `type`: `coordinacion` (punto de coordinación), `control` (punto de control), `contacto` (punto de contacto), `comprobacion` (punto de comprobación), `conexion` (punto de conexión). `text` es la identificación que va dentro del símbolo (p. ej. «PC1», «A», «2»).
- **Líneas** → `LineString` con `properties.mcc`: `{"type":"limitecombate"}`. `type`: `limiteelementos` (límite entre elementos), `limitecombate` (límite de combate), `via` (vía de infiltración), `camino` (camino). Añada `"color"` y `"weight"` si la imagen los muestra.
- **Zona** → `Polygon` con `properties.mcc`: `{"type":"zona","limiteArc":0.5,"echelon":"battalion"}` (`echelon` como en unidades).

### A.7 Líneas defensivas → `LineString` con `properties.defensiveLine`

`{"color":"#C62828","weight":3,"defensiveLine":{"tipo":"zanja"}}`. `tipo`: `alambrico` (Obstáculo alámbrico) · `fortificada` (Línea fortificada) · `zanja` (Zanja antitanque) · `puntofuerte` (Punto fuerte) · `obstaculosgenerales` (Obstáculos generales).

### A.8 Límites con escalón → `LineString` con `properties.limite`

`{"color":"#000000","weight":3,"limite":{"enabled":true,"echelon1":"brigade","echelon2":"battalion"}}` (escalones como en unidades: son las marcas X, XX, II… que se dibujan sobre el límite; `echelon2` puede ir vacío).

---

## ANEXO B · ELEMENTOS COMPLEJOS: USE EL MÉTODO DE LA PLANTILLA

Estos elementos tienen muchos parámetros internos (anchos, proporciones, ángulos, textos en varias posiciones). **No los invente a mano**: la plataforma los exporta con su estructura exacta y eso es lo que hay que imitar.

**Elementos que requieren plantilla:**
- **Flechas** (flecha común, `Dirección de ataque`, `Eje de avance`, avenidas de aproximación aéreas).
- **Tareas tácticas** (bloquear, canalizar, despejar, penetrar, desarticular, destruir, interdictar, neutralizar, retener, contener, fijar, ocupar, asegurar).
- **MCC aéreas**: `a_pca` (Punto de Control Aéreo) · `a_pll` (Punto de llamada) · `a_piaa` (Punto Inicial de Apoyo Aéreo) · `a_pcc` (Punto de control de las comunicaciones) · `a_pt` (Punto Terminal (Actividades aeromóviles)) · `a_corredor` (Corredor Aéreo) · `a_rumbo` (Línea de rumbo) · `a_zor` (Zona de Operaciones Restringida) · `a_zvp` (Zona de vuelo prohibido) · `a_zvr` (Zona de vuelo restringido).
- **MCC de Apoyo de Fuego**: `f_afr` (Área de Fuego Restringido) · `f_anf` (Área de No Fuego) · `f_zf` (Zona de Fuego) · `f_lcsf` (Línea de coordinación y seguridad de los fuegos) · `f_lcaf` (Línea de coordinación del apoyo de fuego) · `f_lcf` (Línea de Coordinación de Fuego) · `f_oo` (Línea O-O) · `f_conc` (Concentración) · `f_gf` (Grupo de fuegos) · `f_sf` (Serie de fuegos) · `f_blanco` (Blanco lineal) · `f_lzv` (Límite de Zona Vital) · `f_lpv` (Límite de Punto Vital) · `f_lam` (Línea de alcance máximo) · `f_ldm` (Línea de detección mínima) · `f_leval` (Línea de Evaluación) · `f_ldestr` (Línea de destrucción) · `f_pcc` (Punto de coordinación de las comunicaciones) · `f_pn` (Punto de notificación) · `f_obs` (Observador) · `f_oaa` (Observador Adelantado de Artillería) · `f_osn` (Observador/Sensores).
- **MCC Logísticas** (tipos `l_...`): lugares de reunión, intercambio, distribución, reabastecimiento y efectos por clase.
- **Ingenieros** (pasajes, campos minados, obstáculos), **línea ofensiva** y herramientas de la barra **Táctica**.
- Imágenes superpuestas.

**Método (3 pasos, los hace quien tiene acceso a la plataforma):**
1. En la pestaña **Edición**, cree una capa de prueba y dibuje **un ejemplo de cada tipo** que aparezca en la imagen (una flecha, una tarea de cada clase, una MCC de cada clase, etc.) con la misma herramienta de la barra. Ajuste los textos de ejemplo para ver dónde va cada dato.
2. Use **«⇩ Exportar capa activa»** (sección Importar / Exportar). Se descarga un archivo `.geojson`.
3. Adjunte ese archivo a la IA **junto con la imagen** y agregue esta instrucción:

> *«Adjunto un GeoJSON exportado por la plataforma con un ejemplo de cada elemento complejo. Para cada elemento equivalente de la imagen, copie la estructura de `properties` del ejemplo del mismo tipo, **sin quitar ni renombrar ningún campo**, y cambie solo lo que corresponde: las coordenadas (en `geometry` y, en las flechas, también `tail`, `head` y `curve`, que van como `[latitud, longitud]`), los textos y la afiliación. Conserve el resto de valores del ejemplo.»*

**Regla para la IA:** si el usuario adjunta un GeoJSON de ejemplo exportado por la plataforma, ese archivo **manda** sobre cualquier otra descripción de este documento. Si un elemento de la imagen no tiene ejemplo en la plantilla, **no lo invente**: déjelo fuera del GeoJSON y anótelo en la tabla de dudas como «requiere plantilla».

> Atención al orden de coordenadas: en `geometry` va `[longitud, latitud]`; dentro de `properties.arrow` (`tail`, `head`, `curve`) va `[latitud, longitud]`.
