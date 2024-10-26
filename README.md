Claro, aquí tienes el README completo con todos los archivos que has proporcionado:

---

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
  ```

### Script `test.sh`

```bash
#!/bin/bash -x
#
# USAGE: ./test.sh <nameserver-ip>
#

# Salir si algún comando falla
set -euo pipefail

function resolver () {
    dig $nameserver +short $@
}

nameserver=@$1

resolver mercurio.sistema.test
resolver venus.sistema.test
resolver tierra.sistema.test
resolver marte.sistema.test

resolver ns1.sistema.test
resolver ns2.sistema.test

resolver sistema.test mx

resolver sistema.test ns

resolver -x 192.168.57.101
resolver -x 192.168.57.102
resolver -x 192.168.57.103
resolver -x 192.168.57.104
```

## Archivos del Proyecto

### `Vagrantfile`

```ruby
Vagrant.configure("2") do |config| 
  config.vm.box = "debian/bookworm64"

  config.vm.define "master" do |master|
    master.vm.hostname = "tierra.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", name: "update", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y bind9
      cp -v /vagrant/named /etc/default/named
      sudo systemctl restart named
      cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
      cp -v /vagrant/tierra_files/named.conf.local /etc/bind/named.conf.local
      cp -v /vagrant/tierra_files/sistema.test.dns /var/lib/bind/sistema.test.dns
      cp -v /vagrant/tierra_files/57.168.192.in-addr.arpa.dns /var/lib/bind/57.168.192.in-addr.arpa.dns
      cp -v /vagrant/tierra_files/resolv.conf /etc/resolv.conf
      sudo systemctl restart bind9
    SHELL
  end

  config.vm.define "dnslave" do |dnslave|
    dnslave.vm.hostname = "venus.sistema.test"
    dnslave.vm.network "private_network", ip: "192.168.57.102"
    dnslave.vm.provision "shell", name: "update", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y bind9
      cp -v /vagrant/named /etc/default/named
      sudo systemctl restart named
      cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
      cp -v /vagrant/venus_files/named.conf.local /etc/bind/named.conf.local
      sudo systemctl restart bind9
    SHELL
  end
end
```

### `named.conf.options`

```plaintext
acl "redesPermitidas" {
    127.0.0.0/8;
    192.168.57.0/24;
};

options {
    directory "/var/cache/bind";
    
    dnssec-validation yes;

    recursion yes;
    allow-recursion { redesPermitidas; };
    allow-query { redesPermitidas; };

    forwarders {
        208.67.222.222;
    };
    
    forward only;

    listen-on { any; };
    listen-on-v6 { any; };
};
```

### `named`

```plaintext
# 
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"
```

### `.gitignore`

```plaintext
.vagrant/
*~
```

### `tierra_files/57.168.192.in-addr.arpa.dns`

```plaintext
;
; Zona inversa para 192.168.57.0/24
;
$TTL 86400
@ IN SOA tierra.sistema.test. admin.sistema.test. (
    2          ; Serial
    3600       ; Refresh
    1800       ; Retry
    604800     ; Expire
    7200       ; Negative Cache TTL
)
;
@ IN NS tierra.sistema.test.
103 IN PTR tierra.sistema.test.
102 IN PTR venus.sistema.test.
101 IN PTR mercurio.sistema.test.
104 IN PTR marte.sistema.test.
```

### `tierra_files/named.conf.local`

```plaintext
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.test" {
    type master;
    file "/var/lib/bind/sistema.test.dns";
    allow-transfer { 192.168.57.102; };
    allow-query { 127.0.0.0/8; 192.168.57.0/24; };
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/var/lib/bind/57.168.192.in-addr.arpa.dns";
    allow-transfer { 192.168.57.102; };
    allow-query { 127.0.0.0/8; 192.168.57.0/24; };
};
```

### `tierra_files/sistema.test.dns`

```plaintext
;
; sistema.test
;
$TTL 86400
@ IN SOA tierra.sistema.test. admin.sistema.test. (
    2          ; Serial
    3600       ; Refresh
    1800       ; Retry
    604800     ; Expire
    7200      ; Negative Cache TTL
)
;
@ IN NS tierra.sistema.test.
sistema.test. IN  A   192.168.57.103
tierra.sistema.test. IN A 192.168.57.103
venus.sistema.test. IN A 192.168.57.102
mercurio.sistema.test. IN A 192.168.57.101
marte.sistema.test. IN A 192.168.57.104

; Alias
ns1.sistema.test. IN CNAME tierra.sistema.test.
ns2.sistema.test. IN CNAME venus.sistema.test.

mail.sistema.test. IN CNAME marte.sistema.test.

; Registro MX (servidor de correo)
@ IN MX 10 marte.sistema.test.
```

### `venus_files/named.conf.local`

```plaintext
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.test" {
    type slave;
    file "/var/lib/bind/sistema.test.dns";
    masters { 192.168.57.103; };
    allow-query { 127.0.0.0/8

; 192.168.57.0/24; };
};

zone "57.168.192.in-addr.arpa" {
    type slave;
    file "/var/lib/bind/192.168.57.rev";
    masters { 192.168.57.103; };
    allow-query { 127.0.0.0/8; 192.168.57.0/24; };
};
```

## Conclusiones

La práctica ha demostrado la capacidad de establecer un entorno DNS funcional y escalable con configuraciones adecuadas para la transferencia de zonas, registros MX y alias. Al ejecutar el script `test.sh`, se validan todas las configuraciones y se asegura que el sistema responde correctamente a las consultas DNS.
