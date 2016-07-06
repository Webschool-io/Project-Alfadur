# Project-Alfadur

Alfadur é o Deus supremo na mitologia nórdica. Embora muitas vezes chamado de Odin, Alfadur (que significa todo-pai) é representado pelo Deus incriado e infinito.
De acordo com a lenda, após o Ragnarok, Alfadur seria responsável por recriar o mundo.

![](https://github.com/Webschool-io/Project-Alfadurff/blob/master/alfadur.jpg)


Um padrão/framework/motherfucker de Styleguide/Frontend Driven Development com Atomic Design Behavior para a criação de componentes isomórficos que isolem as responsabilidades do designer, desenvolvedor, UXer.

Onde teremos um padrão de nomenclatura para que os átomos, moléculas e organismos possam ser reaproveitados em qualquer outro sistema, esses contendo seus comportamentos e o seu visual sendo gerao a partir das definições no styleguide.

## Prova de Conceito

### View

```html
<form action="/api/pessoas/fisicas">
  <input class="atom-input-nome" type="text" name="nome">
  <input class="atom-input-cpf" type="text" name="cpf">
  <button type="submit">Cadastrar</button>
</form>
```

Vamos pegar o *input* de CPF e traduzir ele para POJO (Plain Old JavaScript Object):

```js
const Atom = {
  name: "atom-cpf",
  type: "input/text",
  events: [
    {
      name: "blur",
      callback: isCPF
    },
    {
      name: "success",
      callback: isCPFSuccess
    },
    {
      name: "error" ,
      callback: isCPFError
    }
  ]
}
```

Então perceba que não precisamos adicionar a chamada das funções nos eventos diretamente no HTML pois podemos fazer com [addEventListener](https://developer.mozilla.org/pt-BR/docs/Web/API/Element/addEventListener) e [querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) algo assim:

```js
const atom = Atom.name
const atoms = document.querySelectorAll(atom)
atoms.forEach((el) => Atom.events.forEach((evt) => el.addEventListener(evt.name, evtcallback, false); ))
```

E todas as funções utilizadas pelos átomos são módulos atômicos já existentes que serão apenas importadas, por exemplo:


```js
// isName
'use strict';

module.exports = (value) => {
  const isEmpty = require('./isEmpty')(value);
  const isString = require('./isString')(value);

  if(isEmpty) return false;
  if(!isString) return false;

  return (value.length > 3 && value.length < 80);
}
```

Que irá usar dois outros Quarks:

- isEmpty
- isString

```js
// isString
'use strict';

module.exports = (value) => {
  if (typeof value === 'string' || value instanceof String) return true;
  return false;
}
```

```js
// isEmpty
'use strict';

module.exports = (value) => {
  const isNull = (value === null);
  const isUndefined = (value === undefined);
  const isEmpty = (value === '');
  if (isNull || isUndefined || isEmpty) return true;
  return false;
}
```


```js
// isCPF
'use strict';

module.exports = (value) => {

  const isEmpty = require('../isEmpty/isEmpty')(value);
  if(isEmpty) return false;

  const isString = require('../isString/isString')(value);
  if(!isString) return false;


  let campos = value.split('');
  let colunas = [];
  
  const arrayPrimeiroDig = ['10','9','8','7','6','5','4','3','2'];
  const primeiroDig = 2;

  const arraySegundoDig = ['11','10','9','8','7','6','5','4','3','2'];
  const segundoDig = 1;

  if(value.length != 11) return false;
  if(value == '00000000000') return false;


  const verificaDigito = (campos , colunas , arrayVerificador,numDigito) => {   
    
    for(let i = 0; i < arrayVerificador.length; i++) {
      colunas[i] = campos[i] * arrayVerificador[i];
    }

    let soma = 0;

    colunas.forEach((valor) => {
      soma = valor + soma;
    });

    let resto = soma % 11;
    let digitoVerificador = null;

    if(resto < 2) {
      digitoVerificador = 0;
    } else {
      digitoVerificador = 11 - resto;
    }

    let digito = campos.length - numDigito;

    if(digitoVerificador == campos[digito])
      return true;
    return false;
  }

  let valorPrimeiro = verificaDigito(campos,colunas,arrayPrimeiroDig,primeiroDig);
  let valorSegundo = verificaDigito(campos,colunas,arraySegundoDig,segundoDig);

  if(valorPrimeiro && valorSegundo) 
    return true;
  return false;

}
```

[Quark - isCpf](https://github.com/Webschool-io/Node-Atomic-Design_QUARKS/blob/master/isCpf/isCpf.js)

Agora imagine que cada função irá emitir um evento dependendo do retorno de cada função.

> - Por quê?

Para que nosso formulário possa ser uma Molécula que não precise conhecer seus filhos, pois os mesmos se auto-validarão emitindo eventos de erro da validação com sua mensagem de erro para que o formulário só libere o evento de `submit` quando não tiver recebido **nenhum** evento de erro.

O próprio botão de `submit` só será habilitado após o formulário emitir o evento de sucesso.

Vamos ver como seria o caminho dos dados nessa arquitetura, lembrando que todas funções assíncronas utilizarão Promises.

**IDA no frontend:**

```
atom-input -> valida -> emite evento -> molecule-form escuta eventos -> emite evento caso não tenha recebido nenhum evento de erro -> habilita atom-submit -> emite evento que envia o objeto do form para um service -> que envia requisição HTTP para API
```

**IDA no backend:**

```
rota recebe requisição -> emite evento para o Organismo/Controller -> emite evento para Organela -> emite evento para o DAL (Data Access Layer)
``` 

**VOLTA no backend:**

```
DAL emite evento para serviço de resposta -> Serviço emite resposta HTTP com o status code correto
```

**VOLTA no frontend:**

```
Promise do Serviço recebe resposta, executa o callback correto que irá emitir 1 evento com a resposta para o Model -> Model escuta o evento e se atualiza se necessário
```

## Eventos

Pessoalmente acho interessante que cada módulo que utilize de eventos possua um cache interno com pelo menos os 5 últimos eventos emitidos com um contador, onde o mesmo seja incrementado a cada vez que ele seja escutado e esse contador seja repassado para o próximo evento a ser chamado, qm sabe com algum identificador de qual módulo o escutou, para emularmos "transações" com eventos.

Podemos nos basear no oplog do MongoDb, o qual é utilizado para sincronizar seus servidores.



Vamos utilizar [um exemplo que criei para o Offline-first](http://nomadev.com.br/fullstack-offline-api-first/):

```js
{
  "token": "nomeDoModuloQueEscutouOEvento:TOKEN",
  "timestamp" : Date.now(),
  "ns" : "MeuSistema.nomeDoModulo",
  "event": "",
  "contador": 0,
  "from": "nomeDoModuloQueEmitiuOEvento"
  "data": {}
}
```

Agora vamos imaginar com nosso exemplo do formulário onde o campo de CPF irá emitir um evento para o `form`, onde o mesmo deverá logar o seguinte objeto no seu cache:

```js
{
  "token": "molecule-form:T0k3nD01d3r4",
  "timestamp" : Date.now(),
  "ns" : "Webschool.User",
  "event": "validate",
  "contador": 1,
  "from": "atom-cpf"
  "data": {
    validate: true, 
    obj: {
      name: "cpf", 
      value: "123456789-01"
    }
  }
}
```

Como ele recebeu `{validate: true}` ele irá emitir 1 evento para o botão de submit, gerando esse *oplog* no módulo do botão de submit:

```js
{
  "token": "atom-submit:T0k3nD4z1k444",
  "timestamp" : Date.now(),
  "ns" : "Webschool.User",
  "event": "validate",
  "contador": 2,
  "from": "molecule-form"
  "data": {validate: true}
}
```


Lembrando que esse *oplog* pode ser persistido localmente via Session/LocalStorage

