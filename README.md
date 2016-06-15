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
    {blur: isCPF}, // VALIDATE
    {sucess: isCPFSuccess},
    {error: isCPFError}
  ]
}
```

Então perceba que não precisamos adicionar a chamada das funções nos eventos diretamente no HTML pois podemos fazer com [addEventListener](https://developer.mozilla.org/pt-BR/docs/Web/API/Element/addEventListener) e [querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) algo assim:

```js
const atom = Atom.name
const atoms = document.querySelectorAll(atom)
atoms.forEach((el) => Atom.events.forEach((evt) => el.addEventListener(evt, modifyText, false)))
```



