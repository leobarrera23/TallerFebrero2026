# TallerFebrero2026
# Obligatorio – Taller Servidores Linux  
## Infraestructura NFS Automatizada con Ansible

---

## 1. Objetivo

Implementar una infraestructura mínima, reproducible e idempotente, automatizada con Ansible y versionada en GitHub, integrando:

- NFS Server en CentOS Stream 9  
- Cliente Ubuntu 24.04 con automount (autofs)  
- Servicio systemd en Ubuntu que publique el directorio compartido mediante `python3 -m http.server`  
- Orquestación completa mediante un playbook maestro (`site.yaml`)

---

## 2. Topología

| Hostname   | Sistema              | Rol                               | IP              |
|------------|---------------------|------------------------------------|-----------------|
| bastion    | CentOS 9            | Nodo de control Ansible           | 192.168.10.1    |
| centos01   | CentOS Stream 9     | Servidor NFS (nfs01)              | 192.168.10.11   |
| ubuntu01   | Ubuntu 24.04        | Cliente NFS + Webserver (app01)   | 192.168.10.21   |

Red utilizada: `192.168.10.0/24`

---

## 3. Diagrama Simplificado

            192.168.10.0/24

┌─────────────────────────────────────────┐
│ │
│ bastion (Ansible) │
│ 192.168.10.1 │
│ │ SSH + Ansible │
│ ▼ │
│ centos01 (NFS Server) │
│ 192.168.10.11 │
│ /srv/nfs/shared │
│ │ NFS │
│ ▼ │
│ ubuntu01 (Cliente + Webserver) │
│ 192.168.10.21 │
│ /mnt/shared │
│ http://192.168.10.21:8080
 │
│ │
└─────────────────────────────────────────┘


---

## 4. Requisitos Previos

- Servidores recién instalados.
- Acceso SSH configurado entre `bastion` y los nodos.
- Ansible instalado en `bastion`.
- Inventario configurado en: `inventories/hosts.ini`

Instalar colecciones necesarias:

```bash
ansible-galaxy collection install -r collections/requirements.yaml
5. Estructura del Proyecto
TallerFebrero2026/
│
├── inventories/
│   └── hosts.ini
│
├── playbooks/
│   ├── nfs-server.yaml
│   ├── nfsclient.yaml
│   ├── ubuntu-ufw.yaml
│   └── webserver.yaml
│
├── file/
│   ├── auto.nfs
│   ├── nfs.autofs
│   └── shared-http.service
│
├── collections/
│   └── requirements.yaml
│
└── site.yaml
6. Ejecución
6.1 Ejecución completa de la infraestructura

Para desplegar toda la infraestructura desde cero:

ansible-playbook -i inventories/hosts.ini site.yaml --ask-become-pass

Este playbook maestro ejecuta en orden:

Configuración del servidor NFS (centos01)

Configuración del cliente NFS con autofs (ubuntu01)

Configuración del firewall en Ubuntu

Configuración del servicio web systemd

Los playbooks fueron diseñados para ser idempotentes y ejecutables en servidores recién instalados.

6.2 Ejecuciones individuales de los Playbooks

En caso de requerir pruebas parciales o despliegue por etapas, los playbooks pueden ejecutarse individualmente.

a) NFS Server (centos01)
ansible-playbook -i inventories/hosts.ini playbooks/nfs-server.yaml --ask-become-pass

Configura:

Instalación de nfs-utils

Creación del directorio /srv/nfs/shared

Exportación a la red 192.168.10.0/24

Configuración de firewall (firewalld)

Servicio nfs-server habilitado y activo

b) Cliente NFS con autofs (ubuntu01)
ansible-playbook -i inventories/hosts.ini playbooks/nfsclient.yaml --ask-become-pass

Configura:

Instalación de autofs, nfs-common, python3

Archivos /etc/auto.master.d/nfs.autofs

Archivo /etc/auto.nfs

Timeout configurado

Servicio autofs habilitado y activo

c) Firewall Ubuntu (ubuntu01)
ansible-playbook -i inventories/hosts.ini playbooks/ubuntu-ufw.yaml --ask-become-pass

Configura:

Política default deny incoming

Política default allow outgoing

Allow SSH

Allow TCP 8080

d) Servicio Web systemd (ubuntu01)
ansible-playbook -i inventories/hosts.ini playbooks/webserver.yaml --ask-become-pass

Configura:

Unit file /etc/systemd/system/shared-http.service

Dependencia de red y autofs

Reinicio automático si falla

Servicio habilitado y activo

7. Referencias

Documentación oficial de Ansible
https://docs.ansible.com/

Ubuntu Server Guide
https://ubuntu.com/server/docs

Red Hat Enterprise Linux Documentation – NFS

systemd documentation
https://www.freedesktop.org/software/systemd/man/systemd.service.html

Manuales del sistema:

man nfs

man exports

man exportfs

man autofs

man systemd.service

man ufw
