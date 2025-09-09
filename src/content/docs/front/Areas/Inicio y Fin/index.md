---
title: Inicio y Fin de Operaciones
description: Inicio y Fin de Operaciones
hero:
    tagline: Inicio y Fin de Operaciones
---

import { Steps } from '@astrojs/starlight/components';

## Base de datos

Se agregan 4 tablas nuevas en la Base de Datos esto para mantener una consistencia en los datos sobre cuando aplazo una actividad un agente y las terminales que comparten.

Se debe de utilizar la tabla existente de `EstacionesTrabajoCat` ya que esta tabla representa las Rampas, la cual es necesaria para tener una consistencia de que rampas existen, esto para que en la tabla de `CRMTerminalesEstaciones` se cree una relacion intermedio donde se pueda relacionar que rampas estas asignadas a `X Terminales` y que Terminales estan asociadas a `X Rampas`.


### EstacionesTrabajoCat

Esta tabla representa a las rampas

```sql
CREATE TABLE EstacionesTrabajoCat(
      IDEstacionesTrabajoCat INT PRIMARY KEY IDENTITY(1,1),
      Descripcion VARCHAR(30) NOT NULL,
      ClaveTableros VARCHAR(10) NOT NULL,
      Sucursal INT NOT NULL,
      ServicioExpress BIT
);
``` 

### CRMTerminalesEstaciones

Mantener una consistencia en que Terminales estan asignadas a que `Rampas` y viceversa, la relacion es con la tabla de `EstacionesTrabajoCat`

```sql
CREATE TABLE CRMTerminalesEstaciones (
    Id INT PRIMARY KEY IDENTITY(1,1),
    IdEstacionesTrabajoCat INT NOT NULL,
    IdTerminal INT NOT NULL,

    -- Llaves foráneas para establecer las relaciones
    FOREIGN KEY (IdEstacionesTrabajoCat) REFERENCES EstacionesTrabajoCat(IDEstacionesTrabajoCat) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (IdTerminal) REFERENCES CRMTerminales(IdTerminal) ON DELETE CASCADE ON UPDATE CASCADE
);
```

### CRMTerminal

El proposito de esta tabla es llevar un registro de las terminales que existen en la Sucursal.

```sql
-- Crear la tabla CRMTerminales
CREATE TABLE CRMTerminales (
    IdTerminal INT PRIMARY KEY IDENTITY(1,1),
    NoTerminal INT NOT NULL,
    Descripcion NVARCHAR(255),
    Sucursal INT NOT NULL,
    Estatus BIT DEFAULT 1,
    IdUsuarioAlta INT NOT NULL,
    IdUsuarioBaja INT,
    IdUsuarioModif INT,
    FechaAlta DATETIME DEFAULT GETDATE(),
    FechaBaja DATETIME,
    FechaUltModif DATETIME DEFAULT GETDATE()
);
```

### CRMTecnicoTerminales

El proposito de esta tabla es llevar registro de que tecnicos estan usando `X` terminal. Por ejemplo la `Terminal 1`, puede ser compartida por los Tecnicos `Juan Perez` y `Oscar Lopez`, la funcionalidad de esta tabla es que al consumirla puedes obtener los tecnicos asigndos a una terminal y asi en el Front-End puedes darle la opcion al usuario de seleccionar con que Agente se va a identificar. Esto ayuda a que desde una Terminal puede haber multiples agentes evitando el que se le otorge una terminal a cada uno.

```sql
-- Crear la tabla intermedia CRMTecnicoTerminales
CREATE TABLE CRMTecnicoTerminales (
    Id INT PRIMARY KEY IDENTITY(1,1),
    IdTerminal INT NOT NULL,
    Agente CHAR(10) NOT NULL,
    Estatus BIT DEFAULT 1,
    
    FOREIGN KEY (IdTerminal) REFERENCES CRMTerminales(IdTerminal)
);
```

### VentaDAgenteHistApla

Nos permite tener un registro de los `Tiempos X Pausa` que a realizado un agente en una actividad, es decir las veces que el tenico pauso la actividad asi como tambien cuando realizo los siguientes eventos: `INICIAR, APLAZAR, REINICIAR, COMPLETAR` la actividad. Aunque funciona como Log, a partir de los registros generados en esta tabla es como calculamos los minutos transcurridos en la actividad si el tecnico pauso la operacion.

```sql
CREATE TABLE VentaDAgenteHistApla (
    ID INT,
    Renglon FLOAT,
    RenglonSub INT,
    RID INT NOT NULL IDENTITY(1,1),
    FechaComenzo DATETIME,
    Agente CHAR(10),
    Fecha DATETIME,
    HoraD CHAR(10),
    HoraA CHAR(10),
    Minutos INT,
    Actividad VARCHAR(100),
    Estado VARCHAR(30),
    Comentarios VARCHAR(255),
    Usuario VARCHAR(50),
);
```

### CRMAdminControlOp

Validar quien es controlista en una sucursal o area (HYP/Servicio), tambien por si el usuario es administrador puede ver todas las ordenes.

```sql
CREATE TABLE CRMAdminInicioFin (
    Id INT IDENTITY(1,1) PRIMARY KEY, 
    IdUsuario INT NOT NULL,          
    Sucursal INT NOT NULL,           
    IdArea INT NOT NULL,             
    Estatus BIT DEFAULT 1,
);
```

### CRMConfiguracionInicioFin

```sql
CREATE TABLE CRMConfiguracionInicioFin (
    Id INT IDENTITY(1,1) PRIMARY KEY, 
    Sucursal INT NOT NULL,           
    IdArea INT NOT NULL,             
    Descripcion NVARCHAR(255),     
    Estatus BIT DEFAULT 1,
	MostrarCitas BIT DEFAULT 0,
	MostrarFlujo BIT DEFAULT 0,
);
```


### CRMSituacionOrdenCAT

Este es el catalogo de situaciones que puede tener una orden. `['En proceso de Hojalatería 1', 'Esperando Mecanica', ...]`
```sql
CREATE TABLE CRMSituacionOrdenCAT (
    IdSituacion INT IDENTITY(1,1) PRIMARY KEY,
    Descripcion NVARCHAR(100) NOT NULL,
    Sucursal INT NOT NULL,
    Estatus BIT DEFAULT 1,
);
```

Ejemplo

```sql
INSERT INTO CRMSituacionOrdenCat (Descripcion, Sucursal)
VALUES 
('Esperando Reparación', 0),
('En Espera de Preparación', 0),
('En Espera de Pulido', 0),
('En Espera de Armado', 0),
('En Espera De Pintura', 0),
('En proceso de Hojalatería 1', 0),
('En proceso de Hojalatería 2', 0),
('Esperando Mecánica', 0),
('En Preparación', 0),
('En Pulido', 0),
('En Espera de Control de Calidad', 0),
('En Armado', 0),
('Inicio de Mecánica', 0),
('En Pintura', 0),
('Esperando Pulido', 0);
```

### CRMSituacionesPorTerminal
Unicamente para HYP, permite asociar las situaciones con las terminales por ejemplo `Hojalateria` unicamente puede ver aquellas ordenes cuya situacion sea `En proceso de Hojalatería 1` y 

```sql
CREATE TABLE CRMSituacionesPorTerminal (
    IdTerminal INT,
    IdSituacion INT,
    Sucursal INT NOT NULL,
    PRIMARY KEY (IdTerminal, IdSituacion),
    FOREIGN KEY (IdTerminal) REFERENCES CRMTerminales(IdTerminal),
    FOREIGN KEY (IdSituacion) REFERENCES CRMSituacionOrdenCAT(IdSituacion)
);
```


### CRMTransicionesSituaciones

La terminal de `Hojalateria` al finalizar la actividad unicamente puede asignar la siguiente actividad con la siguiente situacion `En Espera de Preparación`.

```sql
CREATE TABLE CRMTransicionesSituaciones (
    IdTransicion INT IDENTITY(1,1) PRIMARY KEY,  -- Nueva PK autoincremental
    IdTerminal INT NOT NULL,
    IdSituacionActual INT NULL,
    IdSituacionDestino INT NOT NULL,
    Sucursal INT NOT NULL,
    FOREIGN KEY (IdTerminal) REFERENCES CRMTerminales(IdTerminal),
    FOREIGN KEY (IdSituacionDestino) REFERENCES CRMSituacionOrdenCAT(IdSituacion)
);
```

Para poder generar un registro y designar que terminal tiene permitido modificar la situacion de la orden se aplica de la siguiente forma, el `IdTerminal = 1` es la Terminal de `Hojalateria`, al insertar estos registros significa que la Terminal de Hojalateria tiene permitido cambiar la situacion de la orden a las siguientes situaciones. 

```sql
INSERT INTO CRMTransicionesSituaciones (IdTerminal, IdSituacionDestino)
VALUES 
(1, 1),  -- Puede mover a "En proceso de Hojalatería 1"
(1, 2),  -- Puede mover a "En proceso de Hojalatería 2"
(1, 3);  -- Puede mover a "En Espera de Preparación"
```

Sin embargo es posible restringir aun mas la configuracion de la siguiente forma, lo que quiere decir es que si la Orden estaba con la situacion `En Espera de Preparacion` al cambiar el estado de esta orden unicamente estara permitido a `En proceso Hojalateria 1`. Aqui en lugar de aplicar restriccion por Terminal aplicamos una restriccion a cada terminal por la situacion original de la orden.

```sql
INSERT INTO CRMTransicionesSituaciones (IdTerminal, IdSituacionActual, IdSituacionDestino)
VALUES 
-- Solo puede mover de "En Espera de Preparacion" a "En proceso de Hojalatería 1"
(1, 0, 1),  

-- Solo puede mover de "En proceso de Hojalatería 1" a "En proceso de Hojalatería 2"
(1, 1, 2),

-- Solo puede mover de "En proceso de Hojalatería 2" a "En Espera de Preparación"
(1, 2, 3);
```


### VentaDAgenteReproceso

Solo sera utilizada al crear un `reproceso` de una actividad, ya que no se registrara en VentaDAgente sino en esta tabla para no afectar a Intelisis 

```sql
CREATE TABLE VentaDAgenteReproceso (
    ID INT NOT NULL,
    Renglon FLOAT NOT NULL,
    RenglonSub INT NOT NULL,
    RID INT IDENTITY(1,1) PRIMARY KEY,
    Agente CHAR(10),
    Fecha DATETIME,
    HoraD CHAR(5),
    HoraA CHAR(5),
    Minutos INT,
    Actividad NVARCHAR(100),
    Estado NVARCHAR(30),
    Comentarios NVARCHAR(255),
    CantidadEstandar FLOAT,
    FechaConclusion DATETIME,
    CostoActividad MONEY,
    Sucursal INT NOT NULL,
    SucursalOrigen INT NOT NULL,
    SincroID TIMESTAMP,
    SincroC INT,
    DestajoModulo NVARCHAR(20),
    DestajoID INT,
    Avance FLOAT,
    PorcentajeHyP FLOAT,
    Usuario NVARCHAR(10),
    UltimoCambio DATETIME,
    FecProgIni DATETIME,
    FecProgFin DATETIME,
);
```

### Relacion de tablas

Estas tablas estan relacionadas entre si de `One-To-Many` la tabla `CRMTerminal` tiene muchos `CRMTecnicoTerminales`, ya que la tabla de `CRMTerminal` representa a la Terminal en si y la tabla `CRMTecnicoTerminales` representa a cada tecnico.

En la tabla de `CRMTecnicoTerminales` hay un atributo denominado `Agente`, aqui se va a registrar la clave de Agente que tiene en la tabla de `CRMConfiguracion`.

Ejemplo de como crear un registro.

```sql
DECLARE @IdTerminal INT;

-- Insertar un registro en la tabla CRMTerminales
INSERT INTO CRMTerminales (NoTerminal, Descripcion, Sucursal, Estatus, IdUsuarioAlta)
VALUES (1, 'Terminal de Prueba', 0, 1, 1);

-- Obtener el IdTerminal recién generado
SET @IdTerminal = SCOPE_IDENTITY();

-- Insertar un registro en la tabla CRMTecnicoTerminales con el IdTerminal insertado
INSERT INTO CRMTecnicoTerminales (IdTerminal, Agente, Estatus)
VALUES (1, 'JGVA', 1);
```

En el query estamos creando una terminal cuyo numero de terminal sera `1` y asigamos al tecnico cuyo Clave de Agente es `JGVA`, es decir ahora sabemos que existe la terminal 1, que tiene un tecnico asignado y cuyo tecnico asignado es aquel que tiene la clave de agente `JGVA` que viene de la tabla de `CRMConfiguracion`.


En que nos puede servir esto? A continuacion ejecutamos el siguiente query:

```sql
SELECT 
   ct.Agente,
   ct.IdTerminal,
   c.Nombre AS NombreAgente
FROM 
   CRMTecnicoTerminales ct
JOIN 
   CRMConfiguracion c ON ct.Agente = c.Agente
WHERE 
   ct.IdTerminal = (
         SELECT 
            t.IdTerminal
         FROM 
            CRMTecnicoTerminales t
         WHERE 
            t.Agente IN (
               SELECT 
                     c2.Agente
               FROM 
                     CRMConfiguracion c2
               WHERE 
                     c2.Email = @Email
            )
   );
```

Este query nos retorna todos los agentes asignados a una `Terminal` a partir del `Email` del Agente que obtuvimos de la tabla `CRMConfiguracion`. Es decir, si el Agente cuyo email es `jorge@gmail.com` esta asignado a la terminal numero `1` entonces obtendremos todos los agentes asociados a esa terminal, lo que nos puede dar como resultado que sea el unico tecnico asociado a esa terminal o que haya multiples tecnicos asociados a esa terminal.

### Diagrama



![Diagrama de ControlOp](../../../../assets/Areas/Servicio/ControlOp/Inicio-Fin-DB.svg)


## Procedimientos

### Dar de alta una terminal

### Dar de alta una transicion

<Steps>

1. Crear un registro en `CRMSituacionOrdenCat`

   ```sql
    INSERT INTO CRMSituacionOrdenCAT (Descripcion, Sucursal)
    VALUES ('En Espera De Armado', 0) -- Tendra IdSituacion 1
   ```

2. Crear un registro en `CRMTerminales`, en este caso creamos una Terminal para Hojalateria

    ```sql
    -- Crear la tabla CRMTerminales
    INSERT INTO CRMTerminales ( NoTerminal , Descripcion, Sucursal, IdUsuarioAlta)
    VALUES (1, 'HOJALATERIA HYP', 0, 1) ; -- Tendra IdTerminal 1
    ```
3. Crear un registro en `CRMTransicionesSituaciones`, todos los agentes que se encuentren asignados a la terminal de hojalateria podran modificar la situacion de la orden a `En Espera De Armado` cuando finalicen una actividad.
    ```sql
    INSERT INTO CRMTransicionesSituaciones (IdTerminal, IdSituacionDestino)
    VALUES
    (1, 1),  -- Hojalateria puede cambiar la situacion de la orden a "En Espera De Armado"
    ```

</Steps>



### Dar de alta una situacion por terminal


```sql
INSERT INTO CRMSituacionOrdenCAT (Descripcion, Sucursal)
VALUES ('En Espera de Preparación', 0) -- INSERTAR LA SITUACION POR EJEMPLO 'En Espera de Preparacion' y la sucursal que tendra esta situacion '0' Toyota

INSERT INTO CRMSituacionOrdenCAT (Descripcion, Sucursal)
VALUES ('En Espera Hojalateria', 0) -- INSERTAR LA SITUACION POR EJEMPLO 'En Espera Hojalateria' y la sucursal que tendra esta situacion '0' Toyota

SELECT * FROM CRMSituacionOrdenCat;

SELECT * FROM CRMTerminales;

INSERT INTO CRMSituacionesPorTerminal (IdSituacion, IdTerminal, Sucursal)
VALUES (1, 2, 0); -- ('En Espera de Preparacion', 'En Espera Hojalateria', 'Toyota')

INSERT INTO CRMTransicionesSituaciones (IdTerminal, IdSituacionDestino, Sucursal)
VALUES (1, 1, 0) -- ('Terminal Hojalateria', 'En Espera de Preparacion')
```