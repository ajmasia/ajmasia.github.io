---
title: "Enviando notificaciones a Telegram desde tu Qnap"
date: 2021-01-23
draft: false
author: "Antonio José Masiá"
toc: false
categories:
  - Automatización
tags:
  - Telegram
  - Qnap
---

Conocer en tiempo real todo aquello que ocurre en nuestro [servidor NAS](https://www.masqueteclas.com/libro/) es algo que resulta de sumo interés, sobre todo si cuentas con un buen puñado de servicios desplegados, como es mi caso. Intentos de acceso fallido, actualizaciones disponibles, estado de los backups, etc, son muchas de las notificaciones que el servidor puede enviarnos de forma automática y sin apenas configuración alguna. En este post voy a contarte cómo hacer que esas notificaciones te lleguen a [Telegram](https://telegram.org/) en lugar de una cuenta de correo electrónico que es lo más habitual.

Con el paso del tiempo [Telegram](https://telegram.org/) está demostrando que es un servicio realmente sólido y también privado, sobre todo comparándolo por ejemplo con [WhatsApp](https://www.whatsapp.com/). No hay más que ver la [reciente polémica](https://mundo.sputniknews.com/tecnologia/202101091094062174-la-polemica-politica-de-privacidad-de-whatsapp-dispara-las-descargas-de-telegram/) desatada en las últimas semanas debido precisamente a este tema y que ha conseguido que una gran cantidad de usuarios de [WhatsApp](https://www.whatsapp.com/) se estén pasando a [Telegram](https://telegram.org/). Eso es bueno y es malo a la vez, aunque no voy a entra en ello porque daría para otro post 😀 .

Para poder enviar notificaciones desde nuestro [servidor NAS](https://www.masqueteclas.com/libro/) a [Telegram](https://telegram.org/) necesitamos crear un [Bot](https://www.xataka.com/basics/bots-telegram-que-como-funcionan-recomendados-para-empezar) que se encargue de dicha tarea. Aquí surge un pequeño inconvenientes y es que los [Bots](https://core.telegram.org/bots) de [Telegram](https://telegram.org/) no pueden ser privados, por lo que para mantener nuestra notificaciones a salvo tendremos que crear un **grupo privado** al que podremos ir añadiendo todos aquellos [Bots](https://core.telegram.org/bots) de notificaciones que vayamos creando, consiguiendo de esta forma que las notificaciones de los [Bots](https://core.telegram.org/bots) sólo lleguen a este grupo controlado por nosotros mismos. 

## Creando el Bot en Telegram

Para crear un [Bot](https://core.telegram.org/bots) en [Telegram](https://telegram.org/) hemos de abrir una nueva sesión del `@BotFather`, el padre de todos los Bots. Para ello accederemos a él a través del campo de búsqueda de la pestaña Chats de la app. Una vez estemos dentro del [Bot](https://t.me/botfather), podremos comenzar el proceso, siguiendo la siguiente secuencia de pasos:

```
YourUser, [23 Jan 2021 at 13:38:29]:
/newbot

BotFather, [23 Jan 2021 at 13:38:29]:
Alright, a new bot. How are we going to call it? Please choose a name for your bot.

YourUser, [23 Jan 2021 at 13:40:11]:
My server bot

BotFather, [23 Jan 2021 at 13:40:11]:
Good. Now let's choose a username for your bot. It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.

YourUser, [23 Jan 2021 at 13:40:36]:
my_first_bot

BotFather, [23 Jan 2021 at 13:40:36]:
Done! Congratulations on your new bot. You will find it at t.me/************. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.

Use this token to access the HTTP API:
**********************************************
Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
```

Anota todos los parámetros de definición del [Bot](https://core.telegram.org/bots) que acabas de crear ya que tendremos que usarlos para distintos propósitos, en especial el `token` que deberás **mantenerlo siempre en secreto** para evitar abrir agujeros de seguridad

Cómo comentaba al principio del post, los [Bots](https://core.telegram.org/bots) en [Telegram](https://telegram.org/) son públicos por lo que lo que haremos será crear un grupo cuyo único invitado de momento será el [Bot](https://core.telegram.org/bots) que acabamos de crear. Digo de momento porque más adelante podremos ir añadiendo más [Bots](https://core.telegram.org/bots) destinados a comunicaciones, como por ejemplo para las notificaciones de un servidor de [Home Assistant](https://www.home-assistant.io/), etc.

Para ello crearemos un grupo en [Telegram](https://telegram.org/) añadiendo a nuestro [Bot](https://core.telegram.org/bots) como usuario. 

Una vez realizado este paso, cambiaremos la configuración del [Bot](https://core.telegram.org/bots) para que nadie pueda añadirlo a otro grupo, hecho que sería nefasto para la privacidad.

```
YourUser, [23 Jan 2021 at 13:51:18]:
/setjoingroups

BotFather, [23 Jan 2021 at 13:51:18]:
Choose a bot to change group membership settings.

YourUser, [23 Jan 2021 at 13:51:21]:
@my_first_bot

BotFather, [23 Jan 2021 at 13:51:21]:
'Enable' - bot can be added to groups.
'Disable' - block group invitations, the bot can't be added to groups.
Current status is: ENABLED

YourUser, [23 Jan 2021 at 13:51:24]:
Disable

BotFather, [23 Jan 2021 at 13:51:24]:
Success! The new status is: DISABLED. /help
```

Ya por último necesitamos obtener el `chat_id` del grupo que acabamos de crear. Para ello deberemos comenzar a usar el **[API](https://core.telegram.org/api) de [Telegram](https://telegram.org/)**.

Para obtener dicha información hay diversos caminos. Uno de ellos sería hacer la siguiente `request` a través de nuestro navegador:

```bash
https://api.telegram.org/bot<token>/getUpdates
```

La primera respuesta que obtendremos devolverá un resultado vacío ya que el grupo aún no ha tenido actividad por parte del [Bot](https://core.telegram.org/bots):

```json
{"ok":true,"result":[]}
```

Para que se muestre alguna información deberemos de enviar algún tipo de mensaje desde el grupo al [Bot](https://core.telegram.org/bots):

```
YourUser, [23 Jan 2021 at 14:09:02]:
@my_first_bot Hello World!
```

Si ejecutamos de nuevo la `request`, podremos consultar el `chat_id` del grupo en la propiedad `id` del objeto `chat` cuyo tipo es `group`:

```json
chat: {
	id:********,
	title: "My server bot",
	type: "group",
	all_members_are_administrators: true
}
```

Como dato curioso, comentar que el `id` del grupo comenzará con un símbolo `-`, por ejemplo `-218768712`.

Ahora es momento de asegurarse que la privacidad del [Bot](https://core.telegram.org/bots) está activada:

```
YourUser, [23 Jan 2021 at 13:52:37]:
/setprivacy

BotFather, [23 Jan 2021 at 13:52:37]:
Choose a bot to change group messages settings.

YourUser, [23 Jan 2021 at 13:52:39]:
@my_first_bot

BotFather, [23 Jan 2021 at 13:52:40]:
'Enable' - your bot will only receive messages that either start with the '/' symbol or mention the bot by username.
'Disable' - your bot will receive all messages that people send to groups.
Current status is: ENABLED

YourUser, [23 Jan 2021 at 13:52:42]:
Enable

BotFather, [23 Jan 2021 at 13:52:42]:
Success! The new status is: ENABLED. /help
```

De esta forma nos aseguramos que el [Bot](https://core.telegram.org/bots) no recibirá los mensajes que se envíen al grupo.

Es hora de probar que todo funciona y se pueden enviar notificaciones hacia nuestro grupo. Para ello ejecutaremos la siguiente `request` usando el `token` del [Bot](https://core.telegram.org/bots) y el `chat_id` obtenido de nuestro grupo:

```bash
https://api.telegram.org/bot<token>/sendMessage?chat_id=<chat_id>&text=My%20first%20notification!
```

Podremos comprobar cómo nuestra notificación llega al grupo que hemos creado:

```
YourUser, [23 Jan 2021 at 14:09:02]:
@my_first_bot Hello World!

My server bot, [23 Jan 2021 at 14:17:36]:
My first notification!
```

## Configurando el servicio de notificaciones en Notification Center de Qnap

Para poder enviar notificaciones desde el [NAS](https://www.xataka.com/perifericos/poner-nas-casa-ha-sido-mejores-decisiones-tecnologicas-que-he-tomado-toda-mi-vida) deberemos configurar un servicio `SMSC` custom, empleando para ello la `URL` que veníamos usando hasta ahora para enviar notificaciones a través de nuestro [Bot](https://core.telegram.org/bots). La única diferencia es que tendremos que añadir a la `query`, la plantilla que el servicio usa para pasar el mensaje, que en este caso tendría el siguiente formato:

`@@Text@@&user=@@UserName@@&password=@@Password@@&to=@@PhoneNumber@@`

Estos valores los rellenará automáticamente el sistema, obteniendo la información desde el evento generado.

Nos dirigimos a la app [Notification Center](https://www.qnap.com/solution/notification-center/es-es/), concretamente a la pestaña `Cuenta de servicio y sincornización de dispositivos`, concretamente a la opción `SMS`.

{{< figure src="https://res.cloudinary.com/ajmasia/image/upload/q_auto:eco/v1611416742/blog/posts/010/010_image_01.png" >}}

Una vez allí daremos de alta un nuevo servicio SMSC y añadimos un nuevo servicio.

{{< figure src="https://res.cloudinary.com/ajmasia/image/upload/q_auto:eco/v1611416742/blog/posts/010/010_image_02.png" >}}

Seleccionamos la opción `Custom`, le damos el nombre identificativo al servicio y pegamos nuestra `URL` en el campo `Texto de la Plantilla URL`.

```bash
https://api.telegram.org/bot<token>/sendMessage?chat_id=<chat_id>&text=@@Text@@&user=@@UserName@@&password=@@Password@@&to=@@PhoneNumber@@
```

Acuérdate de que el `chat_id` debe de ser el del grupo, y el `token` el obtenido cuando hemos creado el Bot. Pulsamos en el botón `crear` y ya lo tenemos listo:

{{< figure src="https://res.cloudinary.com/ajmasia/image/upload/q_auto:eco/v1611416742/blog/posts/010/010_image_03.png" >}}

Para probar que el servicio funciona, tan sólo hemos de pulsar en el icono del avión, configurar nuestro país e indicar algún número de teléfono, de lo contrario la prueba fallará, **y et voilà** ! ya recibimos notificaciones. El número de teléfono es indiferente. Todos los mensajes llegarán a nuestro grupo.

Para finalizar, tendremos que editar nuestras reglas para indicar que se envíen las notificaciones a través del servicio que acabamos de crear.

Espero que te haya gustado este post y sobre todo que te haya resultado útil.


#### Notas
- Pruebas realizadas con un [NAS](https://es.wikipedia.org/wiki/NAS) [QNAP TS-453B](https://www.qnap.com/es-es/product/ts-453b) y con un [QNAP TS-230](https://www.qnap.com/es-es/product/ts-230). Puedes ver un análisis de este último modelo [aquí](https://www.youtube.com/watch?v=dRTTrVBkv-o)
- Post de referencia: [Creating a private Telegram chatbot | Alex Sarafian](https://sarafian.github.io/low-code/2020/03/24/create-private-telegram-chatbot.html)
- [Imagen](https://unsplash.com/photos/WLOCr03nGr0) de [Luke Chesser (@lukechesser)](https://unsplash.com/@lukechesser) | [Unsplash](https://unsplash.com/) Photo Community
