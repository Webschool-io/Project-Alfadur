# Project-Alfadurff

Um padrão/framework/motherfucker de Styleguide/Frontend Driven Development com Atomic Design Behavior

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





