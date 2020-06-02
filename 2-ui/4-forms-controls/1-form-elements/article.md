# 폼 프로퍼티와 메서드 

폼(form), 그리고 `<input>`같은 컨트롤 요소에는 특별한 프로퍼티와 이벤트가 많습니다.

이 프로퍼티와 이벤트들을 익히고 나면 폼을 다루기가 훨씬 편리해질 겁니다.

## 폼과 요소 탐색하기

폼은 특수한 컬렉션인 `document.forms`의 구성원입니다.

`document.forms`는 이름과 순서가 주어진 '명명된 컬렉션(named collection)'입니다. 이름이나 번호를 이용해 폼에 접근할 수 있습니다.

```js no-beautify
document.forms.my - the form with name="my"
document.forms[0] - the first form in the document
```

폼을 가져온 다음에는 마찬가지로 명명된 컬렉션인 `form.elements`로 폼의 요소를 얻을 수 있습니다.

예시:

```html run height=40
<form name="my">
  <input name="one" value="1">
  <input name="two" value="2">
</form>

<script>
  // 폼 얻기
  let form = document.forms.my; // <form name="my"> element

  // 요소 얻기
  let elem = form.elments.one; // <input name="one"> element

  alert(elem.value); // 1
</script>
```

라디오 버튼처럼 여러 요소가 동일한 이름을 가질 수도 있습니다.

이런 경우 ‘form.elements[name]`는 컬렉션입니다. 

```html run height=40
<form>
  <input type="radio" *!*name="age"*/!* value="10">
  <input type="radio" *!*name="age"*/!* value="20">
</form>

<script>
let form = document.forms[0];

let ageElems = form.elements.age;

*!*
alert(ageElems[0]); // [object HTMLInputElement]
*/!*
</script>
```

탐색 프로퍼티는 태그 구조에 의존하지 않습니다. 폼 내부에서의 깊이와 관계없이 모든 컨트롤 요소는 `form.elements`에서 접근할 수 있습니다.


````smart header="\'하위 폼(subforms)\'처럼 쓰이는 필드셋(fieldset)"
폼은 하나 이상의 `<fieldset>` 요소를 포함할 수 있습니다.  필드셋도 자신의 내부에 있는 폼 컨트롤에 접근할 수 있는 `elements` 프로퍼티를 가지고 있습니다.

예시:

```html run height=80
<body>
  <form id="form">
    <fieldset name="userFields">
      <legend>info</legend>
      <input name="login" type="text">
    </fieldset>
  </form>

  <script>
    alert(form.elements.login); // <input name="login">

*!*
    let fieldset = form.elements.userFields;
    alert(fieldset); // HTMLFieldSetElement

    // 폼과 필드셋 모두 이름으로 input을 구할 수 있습니다.
    alert(fieldset.elements.login == form.elements.login); // true
*/!*
  </script>
</body>
```
````

````warn header="짧은 표기법: `form.name`"
짧은 표기법인 `form[index/name]`으로도 요소에 접근할 수 있습니다.

`form.elements.login` 대신 `form.login`처럼 쓸 수 있는 거죠.

짧은 표기법은 잘 작동하지만 작은 문제가 있습니다. 요소에 접근해 `name`속성을 변경한 후에도 변경 전 이름을 계속 사용할 수 있습니다.(물론 새로운 이름도 사용할 수 있습니다).

예시를 통해 확인해 봅시다.

```html run height=40
<form id="form">
  <input name="login">
</form>

<script>
  alert(form.elements.login == form.login); // true, 동일한 <input>입니다.

  form.login.name = "username"; //  input의 이름을 변경합니다.

  // form.elements에는 name 속성 변경이 반영되었습니다.
  alert(form.elements.login); // undefined
  alert(form.elements.username); // input

*!*
  // form은 새로운 이름과 이전 이름을 모두 인식합니다.
  alert(form.username == form.login); // true
*/!*
</script>
```

form 요소의 이름을 변경하는 일은 드물기 때문에 보통은 문제가 되지 않습니다.

````

## 역참조(Backreference): element.form

모든 요소는 `element.form`으로 폼에 접근할 수 있습니다. 따라서 폼은 모든 요소를 참조하고, 각 요소 또한 역으로 폼을 참조합니다.

그림으로 나타내면 다음과 같습니다.

![](form-navigation.svg)

예시:

```html run height=40
<form id="form">
  <input type="text" name="login">
</form>

<script>
*!*
  // form -> element
  let login = form.login;

  // element -> form
  alert(login.form); // HTMLFormElement
*/!*
</script>
```

## 폼 요소

이제 폼 컨트롤을 살펴봅시다.

### input과 textarea

체크박스의 값은 `input.value` (string) 또는 `input.checked` (boolean)으로 얻을 수 있습니다.

이렇게 말이죠.

```js
input.value = "New value";
textarea.value = "New text";

input.checked = true; // 체크박스나 라디오 버튼에서 쓸 수 있습니다.
```

```warn header="`textarea.innerHTML` 말고 `textarea.value`를 사용하세요."
`<textarea>...</textarea>`이 중첩 HTML로 내부에 값을 가지고 있긴 하지만, textarea에 입력된 값을 얻기 위해  `textarea.innerHTML`로 접근해서는 안 됩니다.

`textarea.innerHTML`은 페이지를 처음 열 당시 초기화된 HTML만을 저장하기 때문에 현재 상태의 값을 구할 수 없습니다.
```

### select와 옵션

`<select>`에는 세 가지 중요한 프로퍼티가 있습니다.

1. `select.options` -- 하위 요소인 `<option>`의 컬렉션,
2. `select.value` -- 현재 선택된 `<option>` 값,
3. `select.selectedIndex` -- 현재 선택된 `<option>`의 번호.

다음 세 가지 방법으로 `<select>`의 값을 설정할 수 있습니다.

1. 해당 `<option>` 요소를 찾아 `option.selected`속성을 `true`로 설정합니다.
2. `select.value`를 원하는 값으로 설정합니다.
3. `select.selectedIndex`를 원하는 옵션 번호로 설정합니다.

첫 번째 방법이 가장 확실하지만 `(2)` and `(3)`이 대체로 더 편리합니다.

예시:

```html run
<select id="select">
  <option value="apple">Apple</option>
  <option value="pear">Pear</option>
  <option value="banana">Banana</option>
</select>

<script>
  // 세 가지 코드의 실행 결과는 모두 같습니다.
  select.options[2].selected = true;
  select.selectedIndex = 2;
  select.value = 'banana';
</script>
```

대부분의 다른 컨트롤 요소와 달리 `<select>`는 `multiple` 속성이 있으면 옵션을 다중 선택할 수 있습니다. 거의 쓰이지 않는 기능이지만 필요한 경우 `<option>`에서 `selected` 속성을 추가·제거하는 첫 번째 방법으로 값을 설정해야 합니다. 

다중 선택한 옵션의 컬렉션은 다음 예시처럼 `select.options`로 가져올 수 있습니다. 

```html run
<select id="select" *!*multiple*/!*>
  <option value="blues" selected>Blues</option>
  <option value="rock" selected>Rock</option>
  <option value="classic">Classic</option>
</select>

<script>
  // 다중 select에서 선택된 모든 값을 가져옵니다.
  let selected = Array.from(select.options)
    .filter(option => option.selected)
    .map(option => option.value);

  alert(selected); // blues,rock  
</script>
```

`<select>`의 전체 명세서는 아래 링크에서 볼 수 있습니다 <https://html.spec.whatwg.org/multipage/forms.html#the-select-element>.

### new Option

new Option 생성자는 잘 사용되지는 않지만 흥미로운 점이 있습니다.

[명세서](https://html.spec.whatwg.org/multipage/forms.html#the-option-element)에 `<option>` 요소를 생성하는 간단하고 멋진 문법이 있습니다.

```js
option = new Option(text, value, defaultSelected, selected);
```

매개변수:

- `text` -- option 내부의 텍스트,
- `value` -- option의 값,
- `defaultSelected` -- `true`로 설정하면 `selected` HTML 속성이 생성됨,
- `selected` -- `true`로 설정하면 해당 옵션이 선택됨.

`defaultSelected`와 `selected`의 역할에 약간의 혼동이 있을 수 있습니다. `defaultSelected`는 HTML 속성을 설정해 `option.getAttribute('selected')`를 사용할 수 있도록 합니다. `selected` 는 옵션을 선택할 것인지 말 것인지를 정합니다. 대체로 두 값 모두 `true`로 설정하거나 둘 다 설정하지 않습니다(설정하지 않으면 `false`를 설정한 것과 같습니다).

예시:

```js
let option = new Option("Text", "value");
// <option value="value">Text</option> 가 생성됩니다.
```

이번엔 같은 요소를 선택된 상태로 생성합니다.

```js
let option = new Option("Text", "value", true, true);
```

Option 객체에는 다음과 같은 프로퍼티가 있습니다.

`option.selected`
: option의 선택 여부.

`option.index`
: `<select>`에 속한 옵션들 사이의 순서를 나타내는 번호.

`option.text`
: 사용자에게 보일 옵션의 텍스트 내용.

## References

- 명세서: <https://html.spec.whatwg.org/multipage/forms.html>.

## 요약

폼 탐색하기

`document.forms`
: `document.forms[name/index]`로 폼에 접근할 수 있습니다.

`form.elements`  
: 폼 요소는 `form.elements[name/index]` 또는 `form[name/index]`로 접근합니다. `elements` 프로퍼티는 `<fieldset>`에도 똑같이 작동합니다.

`element.form`
: 요소는 `form` 프로퍼티에서 자신이 속한 폼을 참조합니다.

각 요소의 값은 `input.value`, `textarea.value`, `select.value` 등으로 접근할 수 있습니다. 체크박스와 라디오 버튼에서는 `input.checked`를 사용할 수 있습니다.

`<select>`에서는 `select.selectedIndex`의 인덱스나 `select.options` 컬렉션의 옵션을 통해 값을 구할 수도 있습니다.

지금까지는 폼을 다루기 위한 기초적인 내용이었습니다. 이 튜토리얼에서 앞으로 더 많은 예시를 만날 것입니다.

다음 챕터에서는 어느 요소에서든 발생할 수 있지만 대부분 폼에서 처리되는 `focus`와 `blur` 이벤트를 다루겠습니다. 
