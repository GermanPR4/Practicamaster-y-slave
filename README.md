# Práctica DNS: Configuración Maestro-Esclavo

Este proyecto consiste en la configuración de un servidor DNS con una arquitectura Maestro-Esclavo para el dominio `sistema.test`. Utiliza dos máquinas virtuales creadas mediante Vagrant: una como servidor maestro (`tierra.sistema.test`) y otra como servidor esclavo (`venus.sistema.test`). Además, incluye zonas directas e inversas, así como la transferencia de zonas entre los servidores.

## Requisitos del Proyecto

- **GitHub Repository**: Incluye un repositorio con commits realizados de forma gradual, un `.gitignore` que excluye directorios como `.vagrant` y archivos de respaldo.
- **Vagrantfile**: Para la creación de las máquinas virtuales necesarias.
- **Configuración DNS**: Se incluye una configuración detallada para los servidores maestro y esclavo, con alias, registros A y MX, entre otros.
- **Licencia**: El proyecto se distribuye bajo la licencia GNU GPL v3 o Creative Commons Attribution License, versión 4.0 o posterior.

## Red y Equipos

La red de la práctica está configurada en el rango `192.168.57.0/24`. A continuación, se detallan los equipos utilizados:

| Equipo            | FQDN                | IP              |
|-------------------|---------------------|-----------------|
| Maestro (Debian)   | tierra.sistema.test  | 192.168.57.103  |
| Esclavo (Debian)   | venus.sistema.test   | 192.168.57.102  |
| Alias NS1          | ns1.sistema.test     | Alias de `tierra.sistema.test` |
| Alias NS2          | ns2.sistema.test     | Alias de `venus.sistema.test`  |

## Configuración DNS

### Servidor Maestro (`tierra.sistema.test`)
- Escucha activada sólo para IPv4.
- Opción `dnssec-validation` configurada en `yes`.
- Consultas recursivas permitidas únicamente para las redes `127.0.0.0/8` y `192.168.57.0/24`.
- Control sobre las zonas directa e inversa del dominio `sistema.test`.
- Respuestas negativas en caché por 2 horas (7200 segundos).
- Reenvío de consultas no autorizadas a OpenDNS (`208.67.222.222`).
- Alias configurados:
  - `ns1.sistema.test` → Alias de `tierra.sistema.test`.
  - `ns2.sistema.test` → Alias de `venus.sistema.test`.
  - `mail.sistema.test` → Alias de `marte.sistema.test`.

### Servidor Esclavo (`venus.sistema.test`)
- Configurado como esclavo, replicando la zona directa e inversa de `tierra.sistema.test`.

## Transferencia de Zona

Se verifica la transferencia de zona entre el servidor maestro y el servidor esclavo, observando los registros AXFR y consultando los logs del sistema. La transferencia automática está habilitada para las zonas directa e inversa.

## Comprobaciones

Para verificar la correcta configuración, puedes utilizar los siguientes comandos con `dig` o `nslookup`:

- **Consultas de registros A**:
  ```bash
  dig @tierra.sistema.test sistema.test
