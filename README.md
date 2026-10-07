# Práctica 3 — Modelo Entidad/Relación: Viveros

**Asignatura:** Administración y Diseño de Bases de Datos  
**Grado:** Ingeniería Informática — ULL

---

## Modelo Entidad/Relación

El diagrama E/R del sistema de gestión de viveros de Tajinaste S.A. se encuentra en los archivos `modelo_viveros.drawio` y `modelo_viveros.png` adjuntos en este repositorio.

---

## Entidades

### VIVERO
Representa cada uno de los viveros que forman la red de Tajinaste S.A.

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `codigo_vivero` | Entero positivo, clave primaria | 101 |
| `nombre` | Cadena de texto, máx. 100 caracteres | "Vivero Norte" |
| `localidad` | Cadena de texto | "Santa Cruz de Tenerife" |
| `georeferenciacion` | Atributo compuesto | — |
| `georeferenciacion.latitud` | Decimal | 28.4636 |
| `georeferenciacion.longitud` | Decimal | -16.2518 |

---

### ZONA (entidad débil)
Representa cada una de las zonas en las que se divide un vivero (exterior, almacén, etc.). Es una entidad débil — no puede existir sin el vivero al que pertenece. Se identifica por su `codigo_zona` junto con el vivero del que depende.

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `codigo_zona` | Entero positivo, identificador parcial | 1 |
| `tipo` | Cadena de texto | "exterior", "almacén", "invernadero" |
| `georeferenciacion` | Atributo compuesto | — |
| `georeferenciacion.latitud` | Decimal | 28.4640 |
| `georeferenciacion.longitud` | Decimal | -16.2520 |

> **Restricción semántica:** `codigo_zona` solo identifica a una zona dentro de su vivero. Dos zonas de distintos viveros pueden tener el mismo `codigo_zona`.

---

### PRODUCTO
Representa cada uno de los productos que la empresa vende (plantas, productos de jardinería, decoración).

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `codigo_producto` | Entero positivo, clave primaria | 201 |
| `nombre` | Cadena de texto, máx. 100 caracteres | "Ficus Benjamina" |
| `tipo` | Cadena de texto | "planta", "decoración", "herramienta" |
| `precio` | Decimal positivo, en euros | 12.50 |

---

### EMPLEADO
Representa a cada uno de los trabajadores de Tajinaste S.A., que pueden ser destinados a distintos viveros según la época del año.

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `dni` | Cadena de texto, clave primaria | "12345678A" |
| `nombre` | Cadena de texto, máx. 100 caracteres | "Laura Martín" |

---

### CLIENTE
Representa a los clientes de la empresa. Pueden ser clientes normales o pertenecer al programa de fidelización Tajinaste Plus.

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `codigo_cliente` | Entero positivo, clave primaria | 401 |
| `nombre` | Cadena de texto, máx. 100 caracteres | "Pedro Suárez" |
| `fidelizacion` | Booleano | true (Tajinaste Plus) / false (normal) |

> **Restricción semántica:** Solo los clientes con `fidelizacion = true` tienen pedidos registrados en el sistema y reciben bonificaciones.

---

### PEDIDO
Representa cada pedido realizado por un cliente Tajinaste Plus, gestionado por un empleado responsable.

| Atributo | Dominio | Ejemplo |
|---|---|---|
| `codigo_pedido` | Entero positivo, clave primaria | 501 |
| `fecha` | Fecha (DATE) | 2024-03-15 |
| `importe` | Decimal positivo, en euros | 87.50 |

---

## Relaciones

### CONTIENE — VIVERO y ZONA
Un vivero se divide en varias zonas, pero cada zona pertenece a un único vivero. Es la relación de dependencia que define a `ZONA` como entidad débil.

**Cardinalidad:** 1:N (un vivero contiene muchas zonas)  
**Participación total:** toda zona pertenece obligatoriamente a un vivero.

---

### ASIGNA — ZONA y PRODUCTO
Los productos se asignan a zonas concretas del vivero. Un producto puede estar en varias zonas y una zona puede tener varios productos.

**Cardinalidad:** N:M

| Atributo de la relación | Dominio | Ejemplo |
|---|---|---|
| `cantidad` | Entero no negativo | 50 |

> `cantidad` indica las unidades disponibles de ese producto en esa zona concreta.

---

### DESTINO — EMPLEADO y VIVERO
Los empleados son destinados a un vivero según la época del año. Un empleado nunca puede tener dos destinos simultáneos.

**Cardinalidad:** N:1 (muchos empleados pueden estar en un vivero, pero cada empleado está en uno solo)

---

### TRABAJA — EMPLEADO y ZONA
En cada vivero, el empleado desempeña su tarea en una zona concreta. Se lleva el histórico del puesto y la productividad de cada empleado en cada zona a lo largo del tiempo.

**Cardinalidad:** N:1 (muchos empleados pueden trabajar en una zona, pero cada empleado trabaja en una zona)

| Atributo de la relación | Dominio | Ejemplo |
|---|---|---|
| `puesto` | Cadena de texto | "vendedor", "jardinero", "responsable" |
| `fecha_inicio` | Fecha (DATE) | 2024-01-01 |
| `fecha_fin` | Fecha (DATE), nullable | 2024-06-30 |
| `productividad` | Decimal | 87.5 |

> `fecha_fin` será nulo si el empleado sigue actualmente en esa zona. El histórico se mantiene guardando todos los registros anteriores.

---

### REALIZA — CLIENTE y PEDIDO
Un cliente Tajinaste Plus puede realizar muchos pedidos. Cada pedido pertenece a un único cliente.

**Cardinalidad:** 1:N

> **Restricción semántica:** Solo los clientes con `fidelizacion = true` pueden tener pedidos asociados.

---

### GESTIONA — EMPLEADO y PEDIDO
Cada pedido es gestionado por un único empleado responsable. Un empleado puede gestionar muchos pedidos.

**Cardinalidad:** 1:N

---

### CONTIENE — PEDIDO y PRODUCTO
Un pedido puede incluir varios productos y un mismo producto puede aparecer en muchos pedidos.

**Cardinalidad:** N:M

| Atributo de la relación | Dominio | Ejemplo |
|---|---|---|
| `cantidad` | Entero positivo | 3 |

---

## Restricciones semánticas adicionales

- Un empleado nunca puede tener dos destinos simultáneos — en un momento dado solo está asignado a un vivero.
- Solo los clientes Tajinaste Plus (`fidelizacion = true`) tienen pedidos registrados y reciben bonificaciones.
- Las bonificaciones se calculan en función del volumen de compras mensual, derivado de los pedidos registrados.
- `fecha_fin` en la relación `TRABAJA` debe ser posterior o igual a `fecha_inicio`.
- `cantidad` en cualquier relación no puede ser negativa.
# Practica-3ABDD
