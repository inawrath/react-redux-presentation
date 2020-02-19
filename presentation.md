title: React & Redux
author:
  name: Iván Nawrath
  twitter: inawrath
  url: https://github.com/inawrath
output: index.html
theme: juanbrujo/cleaver-beerjs

--

# React & Redux

## La explicación ✌️ _simple_ ✌️
![react redux](https://cdn-images-1.medium.com/max/800/1*HBoFpeOTCuIDQMKsSpYN7A.png)

--

### _Primero_ que todo

* Esto no es un mantra, ni evangelización ni nada por el estilo.
* No sirve como guia de estilo ni nada, es solo una explicación de como funciona **A GRANDES RASGOS**.
* Si necesita más info, busque la BeerJS más cercana y converse con alguna persona que sepa ReactJS y Redux.
* Algunas tildes fueron omitidas, porque no tengo _~~punta de montaña~~_ idea donde van.
* Existen algunos _typos_ puestos voluntariamente y otros involuntariamente.
* _Pueden esperar en su puesto mientras el codigo funciona correctamente_

--

### ¿Qué es ReactJS?

Es una libreria Javascript enfocada en la visualización, la V del MVC. Nos ofrece grandes beneficios de perfomance, modularidad y promueve un flujo ¿muy claro? de datos y eventos, lo que permite una mejor planeación y desarrollo de apps complejas. ~~¿Alguien dijo Point?~~

--

### ¿Como funciona ReactJS?

El secreto de ReactJS para que funcione TAN bien es su manera en gestionar el DOM. ReactJS implementa algo llama VirtualDOM que en vez de realizar los cambios(_renderizar_) directamente en el DOM, realiza los cambios en el VirtualDOM(Que es una copia identica del DOM) que se encuentra en memoria. Luego de realizar estos cambios, utiliza un algoritmo para comparar las propiedades del VirtualDOM contra las del DOM, y solo aplica los cambios en el DOM de los elementos modificados, no _renderizando_ todo el DOM nuevamente.

--

### Eso suena a mucho trabajo, no?

Si lo comparas con modificar directamente el DOM, esto es mucho mas eficiente, ya que si tenemos una lista de 2000 elementos en la interfaz y ocurren unos 5 cambios, es mas eficiente aplicar estos 5 cambios, ubicar los componentes que tuvieron algun cambio en sus propiedades y renderizar solo estos 5 elementos. Modificar directamente el DOM implica realizar estos 5 cambios y luego renderizar 2000 elementos.

--

### Una mini-app ReactJS (Versión Class Component)

```javascript
import {random} from 'starwars-names'
import React, { Component } from 'react'
import ReactDOM from 'react-dom'

class App extends Component {
  state = {
    name: '[Click the button]'
  }

  getRandomName = () => {
    this.setState({name: random()})
  }

  render() {
    return (
      <div>
        <h1>Random Star Wars Names</h1>
        <div className="name">{this.state.name}</div>
        <hr />
        <button onClick={this.getRandomName}>
          New Random Name
        </button>
      </div>
    )
  }
}

ReactDOM.render(
  <App />,
  document.getElementById('app'),
)
```

[source https://tupo.to/zBUY](https://tupo.to/zBUY)

[execute https://tupo.to/80Qn](https://tupo.to/80Qn)


--

### ¿Qué es Redux?
Basandome en la misma pagina de Redux

> Redux es un contenedor predecible del estado de aplicaciones JavaScript.

No me gusta esa definición, algo más simple seria "el lugar donde dejamos todo lo que queremos guardar si desmontamos todos los componentes y los volvemos a montar."

--

### Los principios de Redux

Redux se basa en tres principios

* Una sola fuente de la verdad
* El estado es de solo lectura
* Los cambios se realizan mediante **funciones puras**

--

* **Una sola fuente de la verdad**
* El estado es de solo lectura
* Los cambios se realizan mediante **funciones puras**

--

### Una sola fuente de la verdad

Todo el estado de tu aplicación esta contenido en un único store, a diferencia de **flux** que se pueden tener multiples contenedores de estado, que si no son manejados de manera correcta, se pueden _sobreescribir_ y perder la información.

Ej.

Si tenemos los modulos `user`, `invoice` y `products` deberia existir un state que contenga 3 indices, para que los estados de cada modulo esten correctamente guardados.
```javascript
{
    user: {...},
    invoice: {...},
    products: {...},
}
```

--

* Una sola fuente de la verdad
* **El estado es de solo lectura**
* Los cambios se realizan mediante **funciones puras**

--

### El estado es de solo lectura

La única forma de modificar el estado es emitir una acción que indique que cambió, es decir, se deben definir de manera explicita que modificaras y cual sera la acción asociada a este cambio.

Ej.

Si quieres dar por pagada una factura y debes cambiar un atributo de `false` a `true` ejecutaras una función `payInvoice()` la cual emitira una acción y sera capturada por el `reducer` correspondiente.

--

#### _HOYGA_, pero que es una acción?

Una acción es un POJO(Plain Old JavaScript Objects), con al menos una propiedad que indica cual es el tipo de acción, y si lo necesitara, otras propiedades que permitan efectuar la acción.

```javascript
{
    type: 'PAY_INVOICE',
    payload: {
        invoice: 154,
    },
}
```

Para que nuestro store realice esta acción se debe _despachar_ la acción, normalmente con la función _**dispatch**_

--

```javascript
const payInvoice = (id) => (dispatch) => {
    dispatch({
        type: 'PAY_INVOICE',
        payload: {
            invoice: id,
        },
    })
}
```
**ó**
```javascript
const payInvoice = (id) => ({
    type: 'PAY_INVOICE',
    payload: {
        invoice: id,
    },
})
```

Las 2 formas son correctas, pero una es utilizada cuando se necesita _despachar_ más de una vez y la otra no.

--

#### (retomando... 😒)

--

* Una sola fuente de la verdad
* El estado es de solo lectura
* **Los cambios se realizan mediante _funciones puras_**

--

### Los cambios se realizan mediante _funciones puras_

Para controlar como el store es modificado por las acciones se usan reducers puros, cada reducer recibe el estado actual y la acción emitida. Esta acción es evaluada dentro del reducer y se ejecuta una copia del estado con las correspondiente modificaciones(si es que corresponde) por parte de la acción emitida.
El antiguo estado es desechado y se coloca el nuevo estado, que tiene lo mismo que él antiguo más las modificaciones.

--
```javascript
const typeAction = {
    PAY_INVOICE: 'PAY_INVOICE',
}
const initialState = []
const invoice = (state = initialState, action) => {
  switch(action.type) {
    case typeAction.PAY_INVOICE:
      return {
        ...state,
        list: state.list.map((one_invoice) => {
          if (action.payload.invoice === one_invoice.id) {
            return {
              ...one_invoice,
              status_pay: true,
            }
          }
          return one_invoice
        })
      }
    default:
      return state
  }
}

export default invoice
```

--

### Oookkk... pero como sabe React que el estado cambio y debe "renderizar" la nueva información?

React y Redux no se comunican "mágicamente". Para que un componente de React pueda leer los estados o emitir las acciones, estás deben ser "inyectadas" al componente, y para esto existe una función de Redux `connect` que nos realiza esta labor.

`connect` recibe un callback `mapStateToProps` y un objeto `mapDispatchToProps` y el componente al cual debe "inyectar" estos elementos.

--

**mapStateToProps** recibe como parametro el estado completo de redux, por ende, solo debemos obtener los valores que necesitamos y retornarlos en la función.

Ej.

```javascript
const mapStateToProps = state => ({
  invoice: state.invoice,
})
```
Esto permitira inyectar un parametro **invoice** al componente con todo los valores que exista en el estado `invoice`

--

**mapDispatchToProps** es un objeto con las acciones que se inyectaran al componente

Ej.

```javascript
import { payInvoice } from 'redux/actions/invoice'

const mapDispatchToProps = {
    payInvoice,
}
```
Esto permitira inyectar la función `payInvoice` al componente y que sea llamada a partir de sus `props` con `this.props.payInvoice()`

--

### Ejemplo completo

```javascript
import { connect } from 'react-redux'

import { payInvoice } from 'redux/actions/invoice'

const mapStateToProps = state => ({
  invoice: state.invoice,
})

const mapDispatchToProps = {
  payInvoice,
}

export connect(mapStateToProps, mapDispatchToProps)(InvoiceComponent)

```

--

### Ok, pero sigo sin entender como es que Redux le dice a React que hay nuevas cosas.

Cada vez que se monta un componente, el metodo connect de redux, deja el callback en un listado de callbacks a ejecutar. Cuando un reducer retorna el nuevo estado, redux realiza un llamado a todos los callbacks que tiene dentro de su listado para inyectar en los props de esos componentes los nuevos valores. Y como React es inteligente, _Reatciona_ a estos cambios y realiza el render según necesidad.

--

![map redux](https://www.sohamkamani.com/4abcf082e1504cd8543a0b45e4dd9c52/final-connect-flow.svg)

--

### Luego compilamos, subimos a producción y todos fe... oh wait!

![Cat](https://img.devrant.com/devrant/rant/r_283073_ic8QB.gif)

--

## Como un pequeño detalle

--

# Sigue sin gustarme el frontend.

--

### Auspiciadores

- Just check the [Cleaver Quick Start](https://github.com/jdan/cleaver). It's so easy!

- [Cleaver - BeerJS Theme](https://github.com/juanbrujo/cleaver-beerjs/).
