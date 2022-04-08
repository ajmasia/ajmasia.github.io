---
title: "Potencia tus formularios con Vuelidate"
date: 2020-03-21
draft: false
author: "Antonio José Masiá"
toc: false
categories:
  - Desarrollo
tags:
  - Vue
  - Formularios
---

Los formularios son uno de los principales medios de entrada de información hacia nuestras aplicaciones web desde el lado del usuario, por lo que deberemos cuidar con mucho detalle cómo funcionan, **tanto por usabilidad como por evitar errores que puedan acabar provocando agujeros de seguridad**.

[Vuelidate](https://vuelidate.js.org/) es una biblioteca para poder hacer validaciones en [Vue.js](https://vuejs.org/), y hoy voy a compartir contigo cómo usarla en tus proyectos.

Tal y como se describe en la [documentación oficial](https://vuelidate.js.org/#getting-started), se trata de una biblioteca ligera **basada en modelos y totalmente desacoplada de la vista**. Soporta el uso tanto de **validadores predefinidos como personalizados**, pudiendo validar información desde distintas fuentes como por ejemplo `getters` de [Vuex](https://vuex.vuejs.org/) o propiedades computadas. Realmente una gran herramienta.


Veamos cómo podemos usarla y cómo funciona:

Si queremos usar [Vuelidate](https://vuelidate.js.org/) de forma global, instalaremos la dependencia en nuestro proyecto y la usaremos a modo de plugin de [Vue.js](https://vuejs.org/):

```bash
# using yarn
yarn add vuelidate

# using npm
npm i vuelidate
```

Inyectaremos la dependencia en la instancia de [Vue.js](https://vuejs.org/)

```js
import Vue from 'vue'
import Vuelidate from 'vuelidate'

Vue.use(Vuelidate)
```

Si queremos usarla de forma puntual en uno de nuestros componentes, sin tener que inyectar la dependencia de forma global en nuestra instancia de [Vue.js](https://vuejs.org/), podemos hacer uso de los mixin de [Vue.js](https://vuejs.org/):

```js
import { validationMixin } from 'vuelidate'

var Component = Vue.extend({
  mixins: [validationMixin],
})
```

Una vez habilitado el plugin, bien de forma global o at través de su mixin, en la instancia de Vue tendremos acceso al modelo `$v` que representa el estado de la validación y mediante el cuál podremos acceder a una serie de valores y métodos para poder gestionar la validación de nuestro formulario.

En terminología técnica de [Vue.js](https://vuejs.org/), $v es una propiedad computada que monitorizar, tanto el estado del formulario como el del modelo.

Además tanto el modelo `data` con el modelo `validations`, quedarán sincronizados.

Las reglas de validación se definirán, previa importación desde `vuelidate/lib/validators`, como atributos dentro del objeto `validations` que definiremos dentro de nuestro componente:

```js
validations: {
  name: { required },
  age: { required, numeric },
},
```

Antes de ver con más detalle cómo funciona el proceso de validación, vamos a analizar el contenido de algunas de las propiedad computada `$v`:


- `$dirty` | Este valor nos indica si el usuario toco el campo o el formulario al menos una vez
- `$invalid` | Este valor muestra el estado de validación del modelo
- `$error` | Valor que nos permite decidir si se muestra o no un error en el formulario
- `$pending` | Indica si algún tipo de validación asíncrona está pendiente de resolverse. Por defecto es siempre `false` para validaciones síncronas
- `$model` | Referencia al valor del modelo validado. `$v.value.$model` es lo mismo que `this.value`

Veamos a hora los métodos:

- `$touch` | Setea de forma recursiva el valor de \$dirty, tanto del modelo como de todos sus elementos secundarios a `true`
- `$reset` | Hace los mismo que `$touch` pero seteando el valor a `false`

Jugando con estos parámetros seremos capaces de controlar nuestro formulario.

Antes de comenzar a implementar la validación de nuestro formulario, deberemos tener clara cuál es la estrategia de validación que usaremos. Por ejemplo *¿validaremos cada campo según el usuario va escribiendo escribe?* *¿Validaremos una vez se pierde el foco del campo?* *¿Validaremos sólo cuando el usuario pulse en botón de envío?* Todo dependerá del tipo de formulario, la cantidad de campos que tenga, etc.

Imaginemos un formulario sencillo como este:

![](https://res.cloudinary.com/ajmasia/image/upload/v1604663974/blog/posts/008/sample_form.png)

```js
<script>
import { validationMixin } from 'vuelidate'
// Import needed vuelidate validators
import { required, numeric } from 'vuelidate/lib/validators'

export default {
  name: 'Form',
  data() {
    return {
      name: '',
      age:  '',
    }
  },
  mixins: [validationMixin],
  validations: {
    name: { required },
    age:  { 
        required, 
        numeric 
    },
  },
  methods: {
    onSubmit() {
      console.log({ name: this.name, age: this.age })
    },
  },
}
</script>
```

Como podemos ver, hemos definido el objeto `validations` mapeando directamente el modelo del formulario e indicando los validadores predefinidos que usará cada entidad del modelo.

La vista estaría definida de la siguiente forma:

```html
<template>
  <b-container>
    <h2>Vue Forms validation with Vuelidate</h2>
    <b-row align-h="center">
      <b-col cols="6">
        <b-form @submit.stop.prevent>
          <b-form-group 
            id="name-group" 
            label="Name"
            label-for="name">
            <b-form-input
              id="name"
              type="text"
              placeholder="Please, enter your name"
              v-model="$v.name.$model"
            ></b-form-input>
          </b-form-group>
          <b-form-group 
            id="name-group" 
            label="Age" 
            label-for="age">
            <b-form-input
              id="suagername"
              type="text"
              placeholder="Please, your surname"
              v-model.number="$v.age.$model"
            ></b-form-input>
          </b-form-group>
          <b-button 
            class="mt-5" 
            variant="primary"
            block @click="onSubmit"
            >Submit</b-button
          >
        </b-form>
      </b-col>
    </b-row>
  </b-container>
</template>
```

Si inspeccionamos el componente `Form` a través de las [Vue devTools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd?hl=en), podremos ver la siguiente estructura:


![](https://res.cloudinary.com/ajmasia/image/upload/v1604663972/blog/posts/008/vue_status.png)

Si queremos deshabilitar el botón `submit` hasta que la validación sea correcta, podemos bindear el atributo `disable` del botón con el atributo de grupo (o global) `$invalid` de `$v`:

```html
<b-button
  class="mt-5"
  variant="primary"
  block
  @click="onSubmit"
  :disabled="$v.$invalid"
  >Submit
</b-button>
```

De esta forma impedimos al usuario que envíe el formulario hasta que este sea válido.

Veamos ahora las validaciones individuales de cada campo:

Podemos hacer la validación de cada campo de dos formas. Bien a la vez que se va escribiendo sobre él o bien cuando se pierde el foco. 

Vemos las dos soluciones:

Validación mientras el usuario escribe:

```html
<b-form-group id="name-group" label="Name" label-for="name">
  <b-form-input
    id="name"
    type="text"
    placeholder="Please, enter your name"
    v-model.lazy="$v.name.$model"
  ></b-form-input>
  
  <b-form-invalid-feedback id="input-2-live-feedback">
    Empty field
  </b-form-invalid-feedback>
</b-form-group>
```

Como podemos observar, estamos usando `bootstrap-vue`. El componente `<b-form-input>` dispone de un atributo denominado `state` que nos permite controlar cuándo se mostrará el mensaje de error definido en el componente `<b-form-invalid-feedback>`. Para que se produzca la validación conforme el usuario va escribiendo en el campo, bindearemos `state` con el estado de validación del formulario:

```html
<b-form-input
    id="name"
    type="text"
    placeholder="Please, enter your name"
    v-model.lazy="$v.name.$model"
    :state="$v.name.$dirty ? !$v.name.$error : null"
></b-form-input>
```

Si observamos, lo que hacemos es verificar si ya el usuario ha usado el campo `$v.name.$dirty `, y en caso afirmativo, chequeamos si hay algún error. De lo contrario `state` será `null`.

Este tipo de validación puede resultar un poco incómoda en los casos en los que el valor válido se obtiene después de que el usuario haya introducido algunos datos, como por ejemplo un mínimo de caracteres o un valor comprendido entre dos números. En este caso lo ideal sería validar al perder el foco. 

Validación al perder el foco: 

Para ello deberemos hacer uso de los métodos `$reset` y `$touch` que nos ofrece vuelidate:

```html
<b-form-input
    id="name"
    type="text"
    placeholder="Please, enter your name"
    v-model.lazy="$v.name.$model"
    :state="$v.name.$dirty ? !$v.name.$error : null"
    @input="$v.name.$reset()"
    @blur="$v.name.$touch()"
></b-form-input>
```

Al detectar el evento `@input` lo que hacemos es resetear el valor `$dirty` del campo y cuando perdemos el foco, `@blur`, seteamos el valor de `$dirty` a true para que vuelidate desencadene la validación que si recuerdas se verificará en el atributo `state`.

Ahora bien, si observamos, el campo `age` es requerido y además debe de ser numérico, por lo que el mensaje de error debería de ser diferente en función de si ha introducido o no algún valor o si el valor introducido es o no numérico. Veamos como podemos resolver esto. 

Una solución podría ser tener distintos `div` en las vista que se mostraran de forma condicional en función del tipo de error que se detecte, aunque esta solución parece poco elegante. Para hacerlo de forma dinámica usaremos una propiedad computada que muestre el mensaje correspondiente en función del tipo de error detectado:


```javascript
computed: {
    getAgeErrorMessage() {
      if (this.$v.age.$error) {
        for (let key in this.$v.age.$params) {
          if (this.$v.age[key] === false) {
            switch (key) {
              case 'required':
                return 'This filed is required'
              case 'numeric':
                return 'The field must be numeric'
              default:
                break
            }
          }
        }
      }
      return null
    },
  },
```

Cada propiedad dentro del modelo contiene un atributo `$params` que contiene la configuración del validador. Lo que estamos haciendo es recorrer dicho atributo y comprobar que si existe algún error vinculado a cada `key`, mostrando el mensaje correspondiente usando un `switch`. 

El lugar de usar un `switch`, podríamos usar un objeto de configuración de los mensajes y devolver aquél que corresponda según el caso. Para ello podemos hacer uso de las `custom properties` que nos aporta Vue.js. 

Definimos dentro de nuestro componente un objeto que contenga todos los mensajes posibles:

```javascript
errorMessages: {
  required: 'This filed is required',
  numeric: 'The field must be numeric',
},
```

Y ahora, en lugar de usar el `switch` llamaremos al mensaje de error correspondiente usando el objeto `$options`:

```javascript
computed: {
    getAgeErrorMessage() {
      if (this.$v.age.$error) {
        for (let key in this.$v.age.$params) {
          if (this.$v.age[key] === false) {
            return this.$options.errorMessages[key]
          }
        }
      }
      return null
    },
  },
```

Parece que está opción es más elegante ¿verdad?

Con [Vuelidate](https://vuelidate.js.org/) podemos hacer un montón de cosas más como por ejemplo [agrupar validaciones](https://vuelidate.js.org/#sub-validation-groups), a [anidarlas](https://vuelidate.js.org/#sub-data-nesting), [agruparlas en colecciones](https://vuelidate.js.org/#sub-collections-validation), definir [esquemas dinámicos](https://vuelidate.js.org/#sub-dynamic-validation-schema) y [parámetros dinámicos](https://vuelidate.js.org/#sub-dynamic-parameters), hacer [validación asíncrona](https://vuelidate.js.org/#sub-asynchronous-validation) e incluso [validaciones retardadas](https://vuelidate.js.org/#sub-delayed-validation-errors). Algunas de estas cosas las veremos en otros post más avanzados.

Espero que te haya resultado útil y recuerda:

> Trata de escribir código limpio, te ayudará a que sea comprensible tanto para tí como para cualquiera que lo lea. **Escribir buen código también es un noble arte**.

Nos vemos en el siguiente post.


