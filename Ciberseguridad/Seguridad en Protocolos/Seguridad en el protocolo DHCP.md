# Seguridad en el protocolo DHCP
# Temas a tratar

- Ataques
	- DHCP Spoofing
		- Definición
		- Cómo realizar el ataque (Kali Linux)
	- DHCP Starvation
		- Definición
		- Cómo realizar el ataque (Kali Linux)
- Mitigaciones
	-  DHCP Snooping (DHCP Spoofing)
		- Configuración de DHCP Snooping en CISCO
	-  Port Security (DHCP Starvation)
		- Configuración de Port Security en CISCO

# Ataques

## DHCP Spoofing

### Definición

>Es un tipo de ataque que afecta al protocolo DHCP, es de tipo Man-In-The-Middle y permite al atacante hacerse pasar por un servidor DHCP legítimo de la subred para otorgar configuraciones maliciosas como una puerta de enlace o servidores DNS maliciosos para derivarlo a robo de credenciales, redireccionamiento a páginas web fraudulentas, entre otros.

### Cómo realizar el ataque (Kali Linux)

>Para ello, emplearemos la herramienta que viene por defecto en Kali Linux "Ettercap", la abriremos mediante el siguiente comando:

```bash
sudo ettercap -G
```

![image](https://github.com/user-attachments/assets/fe3891d4-e8e1-4a57-b141-ec3e79afb70b)


>Aquí, le daremos a la opción de realizar ataque MITM y seleccionaremos DHCP Spoofing:

![image](https://github.com/user-attachments/assets/d6a76622-c0ed-4521-8c66-68edae29a397)

> Una vez le demos, deberemos configurar opcionalmente una pool a brindar, la máscara de subred y un servidor DNS el cual será nuestro equipo de atacante:  

![image](https://github.com/user-attachments/assets/f23d8fa8-03d4-4cee-bd7a-72ee3a06ca17)

>Una vez comiences el ataque, le darás a los clientes la configuración de red maliciosa y podrías, por ejemplo, interceptar contraseñas enviadas mediante formularios HTML que aparecerán en este lugar además de los leases:

![image](https://github.com/user-attachments/assets/0adc78a4-81b4-40d8-8104-76a0c42022bc)
## DHCP Starvation

### Definición

> Es un tipo de ataque que afecta al protocolo DHCP, este ataque permite al atacante obtener masivamente todas las direcciones IP de la pool del servidor DHCP legítimo de la red mediante la suplantación de la dirección MAC. Desencadenando un ataque de Denegación de Servicio (DoS) al hacer que ningún usuario nuevo en la red pueda obtener una dirección IP con la que poder navegar en internet.

### Cómo realizar el ataque (Kali Linux)

>Para realizar el ataque emplearemos la herramienta Yersinia, la instalaremos con:

```bash
sudo apt install yersinia -y
```

>Una vez instalada, la lanzaremos de la siguiente manera:

```bash
sudo yersinia -I
```

> Aquí pulsaremos las teclas "espacio" y "F2" para meternos en el modo DHCP y pulsamos la tecla "x" para ver los ataques disponibles:

![image](https://github.com/user-attachments/assets/176fe693-73d1-49fe-a59b-d5a6fc430eef)

>Pulsaremos la tecla "1" para iniciar el ataque DHCP Starvation y cuando lleve un tiempo pulsaremos "L" y seguidamente "Q" para parar el ataque.

>Si tuvieramos un sniffer activo desde el principio, podríamos ver todos los mensajes DHCPDISCOVER enviados por yersinia:

![image](https://github.com/user-attachments/assets/4913b31b-fc04-41a6-a96c-02759a1978b0)

> Una vez hecho esto, es probable que el servidor DHCP ya no pueda asignar las direcciones IP "gastadas" de la pool a usuarios legítimos, causando una Denegación de Servicio (DoS).
# Mitigaciones

## DHCP Snooping (DHCP Spoofing)

### Configuración de DHCP Snooping en CISCO

>Para poder mitigar un ataque DHCP Spoofing, deberemos configurar el protocolo DHCP Snooping en el switch que conecte a los clientes con el servidor DHCP:

>Primero, configuraremos el DHCP Snooping diciéndole al switch en que VLAN se debe habilitar el protocolo:

```cli-switch-cisco
Switch> enable
Switch# config t
Switch (config)# ip dhcp snooping
Switch (config)# ip dhcp snooping vlan [VID]
```

>Posteriormente, seleccionaremos los puertos que queramos que sean confiables, es decir que no se bloquearán mensajes como DHCPOFFER, DHCPACK o DHCPNAK, cabe destacar que los puertos que no se configure la confianza, por defecto bloquearán los mensajes mencionados anteriormente una vez habilitado el DHCP Snooping:

```cli-switch-cisco
Switch (config)# int [idInterface]
Switch (config-if)# ip dhcp snooping trust
```

## Port Security (DHCP Starvation)

### Configuración de Port Security en CISCO

>Para poder evitar que un atacante adquiera todas las direcciones IP de nuestra pool mediante el ataque DHCP Starvation, deberemos configurar Port-Security en nuestro switch que conecte a los clientes con el servidor DHCP, así lograremos especificar cuandos números de direcciones MAC pueden responder por puerto, evitando que se realice el ataque al falsificar direcciones MAC masivas para obtener toda la pool:

>Primero, accederemos como administradores a la configuración del switch y seleccionaremos la interfaz sobre la que queremos establecer el límite de direcciones MAC:

```cli-switch-cisco
Switch> enable
Switch# config t
Switch (config)# int [idInterface]
```

>Posteriormente, configuraremos el port-security para que, por ejemplo, solo pueda responder una dirección MAC, la cual debe ser la habitual que, en lugar de eliminarla al reiniciarse, se mantendrá y obtendrá de forma dinámica y si se incumple cortará la conexión:

```cli-switch-cisco
Switch (config-if)# switchport port-security
Switch (config-if)# switchport port-security maximum 1
Switch (config-if)# switchport port-security violation shutdown
Switch (config-if)# switchport port-security mac-address sticky
```
