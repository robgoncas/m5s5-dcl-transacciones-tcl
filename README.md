
# Clase: Entendiendo Transacciones con PostgreSQL

## 1. Introducción a las Transacciones en PostgreSQL

### 1.1. ¿Qué es una transacción?
Una **transacción** es una unidad de trabajo que se ejecuta en una base de datos. Está compuesta por una o más operaciones que deben cumplirse en su totalidad para que la transacción se considere exitosa. Si alguna operación falla, la transacción debe revertirse (hacer un **rollback**), dejando la base de datos en su estado original.

*Ejemplo*: Imagina una operación bancaria donde se transfiere dinero de una cuenta a otra. La transacción asegura que si se debita dinero de una cuenta, también se acredita en la otra. Si alguna parte de este proceso falla, ninguna de las cuentas debería verse afectada.

### 1.2. Importancia de las transacciones en las bases de datos
Las transacciones son cruciales para mantener la **integridad** y **consistencia** de los datos. Ayudan a evitar inconsistencias, como en el ejemplo anterior, donde si la transacción falla a la mitad, no se pierde ni se duplica dinero.

## 2. Propiedades ACID

### 2.1. Desglose de las propiedades ACID
Las transacciones deben cumplir con las siguientes propiedades **ACID**:

- **Atomicidad**: Todas las operaciones dentro de una transacción se completan o ninguna lo hace.
- **Consistencia**: Una transacción lleva la base de datos de un estado válido a otro, respetando todas las reglas definidas.
- **Aislamiento**: Las transacciones se ejecutan de manera independiente unas de otras.
- **Durabilidad**: Una vez que una transacción se confirma, sus efectos persisten incluso en caso de fallo del sistema.

### 2.2. Ejemplos prácticos

```sql
-- Ejemplo de una transacción con las propiedades ACID
BEGIN;

-- Atomicidad y Consistencia
INSERT INTO cuentas (id, saldo) VALUES (1, 1000);
INSERT INTO cuentas (id, saldo) VALUES (2, 2000);


-- Transferir 500 unidades de la cuenta 1 a la cuenta 2
UPDATE cuentas 
SET saldo = saldo - 1500 
WHERE id = 1;
AND 
WHERE cuentas.saldo >=1500

UPDATE cuentas SET saldo = saldo + 500 WHERE id = 2;

-- Durabilidad
COMMIT;  -- La transacción se confirma, y los cambios son permanentes.
```

## 3. Data Control Language (DCL)

### 3.1. Introducción a DCL
El **DCL (Data Control Language)** se utiliza para manejar los permisos y accesos a la base de datos. Define quién puede hacer qué dentro de la base de datos.

### 3.2. Comandos DCL
- **GRANT**: Otorga permisos a los usuarios.
- **REVOKE**: Revoca permisos otorgados previamente.

### 3.3. Ejemplo práctico: Crear un usuario y gestionar sus permisos

```sql
-- Crear un nuevo usuario en la base de datos
CREATE USER juan WITH PASSWORD 'password123';

-- Otorgar permisos de SELECT a todas las tablas en el esquema público
GRANT SELECT ON ALL TABLES IN SCHEMA public TO julio_LECTOR;

GRANT SELECT/UPDATE/INSERT ON ALL TABLES IN SCHEMA public TO juan_EDITOR;

GRANT SELECT/UPDATE/DELETE/INSERT ON ALL TABLES IN SCHEMA public TO jose_ADMIN;

-- Revocar permisos a una columan en específico (column-level permissions)
REVOKE SELECT Precio ON Productos TO juan;


-- Verificar los permisos otorgados
--Consola: 
\dp
--pg admin: 
SELECT * FROM pg_roles;


-- Ahora, le damos permiso para insertar datos en una tabla específica
GRANT INSERT ON cuentas TO juan;

-- Revocamos el permiso de SELECT en otra tabla específica
REVOKE SELECT ON transacciones FROM juan;

-- Verificar los permisos después de las modificaciones
\dp cuentas
\dp transacciones
```

*Explicación*: Primero, creamos un usuario llamado "juan" y le otorgamos permisos de lectura (SELECT) sobre todas las tablas en el esquema `public`. Luego, le damos permiso para insertar datos en la tabla `cuentas` y le revocamos el permiso de lectura en la tabla `transacciones`.

## 4. Transaction Control Language (TCL)

### 4.1. Introducción a TCL
El **TCL (Transaction Control Language)** se usa para gestionar las transacciones dentro de la base de datos.

### 4.2. Comandos TCL
- **COMMIT**: Confirma una transacción, haciendo permanentes todos los cambios.
- **ROLLBACK**: Revierta una transacción, deshaciendo todos los cambios realizados desde el último COMMIT.
- **SAVEPOINT**: Crea un punto de control dentro de una transacción, permitiendo un rollback parcial.

### 4.3. Ejemplos prácticos

#### Ejemplo 1: Operaciones básicas de TCL

```sql
-- Iniciar una transacción
BEGIN;

-- Hacer algunos cambios
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

-- Crear un punto de guardado
SAVEPOINT antes_de_actualizar;

-- Realizar otra operación
UPDATE cuentas SET saldo = saldo - 50 WHERE id = 1;

-- Revertir al punto de guardado
ROLLBACK TO antes_de_actualizar;

-- Confirmar la transacción
COMMIT;
```

#### Ejemplo 2: Transacción de pago online con tarjeta de crédito Master Plop

```sql
-- Iniciar la transacción
BEGIN;

-- Paso 3: Registrar la transacción en el historial de pagos
INSERT INTO historial_pagos (id, numero_tarjeta, monto, fecha, estado) 
VALUES (nextval('historial_pagos_seq'), '1234-5678-9012-3456', 150, NOW(), 'Pendiente');

SAVEPOINT verificando_saldo;

-- Paso 1: Verificar el saldo disponible en la tarjeta
SELECT saldo FROM tarjetas WHERE numero_tarjeta = '1234-5678-9012-3456' FOR UPDATE;

-- Paso 2: Si el saldo es suficiente, debitar la cantidad del pago
UPDATE tarjetas SET saldo = saldo - 150 WHERE numero_tarjeta = '1234-5678-9012-3456';

ROLLBACK verificando_saldo;

-- Confirmar la transacción
COMMIT;
```