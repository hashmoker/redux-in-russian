# Предшественники

Redux имеет смешанное наследство. Он схож с некоторыми паттернами и технологиями, но, в то же время, имеет сильные отличия в важных вопросах. Мы раскроем некоторые из этих общностей и различий далее.

### Flux

Может ли Redux считаться [Flux](https://facebook.github.io/flux/) реализацией?

И [да](https://twitter.com/fisherwebdev/status/616278911886884864), и [нет](https://twitter.com/andrestaltz/status/616270755605708800).

(Не беспокойтесь, [создатели Flux](https://twitter.com/jingc/status/616608251463909376) [одобрили Redux](https://twitter.com/fisherwebdev/status/616286955693682688), если это все, что вы хотели знать.)

Redux черпал вдохновление из некоторых важных качеств Flux. Как и Flux, Redux предписывает вам концентрировать вашу логику обновления модели на определенном уровне вашего приложения (“хранилище” в Flux, “редьюсеры” в Redux). Вместо того, чтобы код приложения изменял данные напрямую, оба предлагают вам описывать каждое изменение, как простой объект, называемый "действием".

В отличии от Flux, **Redux не имеет концепции Диспетчера**. Это потому, что он опирается на чистые функции, вместо генераторов событий (event emitters) и чистые функции легко создаются и не требуют дополнительной сущности для управления ими. В зависимости от того, как вы видете Flux, вы можете считать это отклонением или деталью реализации. Flux часто может быть [описан как `(state, action) => state`](https://speakerdeck.com/jmorrell/jsconf-uy-flux-those-who-forget-the-past-dot-dot-dot). В этой трактовке, Redux соответствует архитектуре Flux, но делает ее проще, благодаря чистым функциям.

Другим важным отличием от Flux, является то, что **Redux предполагает, что вы никогда не изменяете ваши данные напрямую.** Вы запросто можете использовать простые объекты и массивы для состояния, но строго рекомендуется изменять их через редьюсеры. Вы всегда должны возвращать новый объект, что проще сделать при помощи [object spread синтаксиса, предлагаемого ES7](https://github.com/sebmarkbage/ecmascript-rest-spread) и внедренного в [Babel](http://babeljs.io) или в библиотеки, типа [Immutable](https://facebook.github.io/immutable-js).

Несмотря на то, что технически *возможно* [писать нечистые редьюсеры](https://github.com/rackt/redux/issues/328#issuecomment-125035516), которые изменяют данные с целью сохранения производительности, мы категорически не советуем вам этого делать. Функциональности, типа time travel, record/replay, или hot reloading сломаются. Кроме того, вряд ли иммутабельность создает проблемы производительности в большинстве реальных приложений, потому что, как показывает [Om](https://github.com/omcljs/om), даже если вы потеряете на создании новых объектов, вы все равно выиграете, избегая дорогостоищих перерендеров и повторных расчетов, т.к. вы точно знаете, что изменилось, благодаря чистоте редьюсера.

### Elm

[Elm](http://elm-lang.org/) - это функциональный язык программирования, вдохновленный Haskell,  созданный [Evan Czaplicki](https://twitter.com/czaplic). Он обеспечивает соблюдение архитектуры [“модель представление обновление” architecture](https://github.com/evancz/elm-architecture-tutorial/), где обновление имеет следующие сигнатуры: `(state, action) => state`. Технически, Elm “обновления” эквивалентны редьюсерам в Redux.

Но, в отличии от Redux, Elm - это язык, так что это дает возможность извлечь выгоду из многих вещей, типа принудительной чистоты, статической типизации, иммутабельности из коробки и сравнения шаблонов (используя `case` выражение). Даже если вы не планируете использовать Elm, вы все равно должны прочесть его документацию и поиграть с ним. Существует интересная [JavaScript библиотека реализующая похожие идеии](https://github.com/paldepind/noname-functional-frontend-framework). Мы должны поизучать его для вдохновения на Redux! Одним из способов, с помощью которого мы можем стать ближе к статической типизации Elm является [использование решений опциональной типизаций, типа Flow](https://github.com/rackt/redux/issues/290).

### Immutable

[Immutable](https://facebook.github.io/immutable-js) - это JavaScript библиотека, внедряющая неизменные структуры данных. Она мощная и имеет своеобразное JavaScript API.

Immutable и большинство похожих библиотек ортогональны Redux. Не стесняйтесь использовать их вместе.

**Redux не важно *как* вы храните состояние — это может быть простым объектом, объектом Immutable или чем-то еще.** Скорее всего Вам понадобится механизм (де)сериализации состояний для написания универсальных приложений (universal apps) и получения их состояния с сервера, но кроме того, вы можете использовать любую библиотеку для сохранения данных *до тех пор, пока она поддерживает иммутабельность*. Например, не имеет смысла использовать Backbone для Redux состояний, поскольку модели Backbone - мутабельны.

Обратите внимание, что даже если ваша иммутабельная библиотека поддерживает курсоры, вы все равно не должны использовать ее в Redux приложении. Все дерево состояний должно рассматриваться как объект доступный только для чтения и вы должны использовать Redux для обновления состояния и для подписки на обновления. Поэтому писать с помощью курсора, не имеет смысла для Redux. **Если вы используется курсоры для выделения дерева состояний из дерева UI и постепенного совершенствования курсоров, вам следует использовать селекторы вместо курсоров.** Селекторы - это комбинируемые геттер-функции. Изучите [reselect](http://github.com/faassen/reselect) для по-настоящему мощной и выразительной реализации комбинируемых селекторов.

### Baobab

[Baobab](https://github.com/Yomguithereal/baobab) - это еще одна популярная библиотека, реализующая иммутабельное API для обновления JavaScript объектов. Несмотря на то, что вы можете использовать ее с Redux, от их совместного использования будет мало толку.

Большинство функциональности Baobab обеспечивается посредством обновления данных через курсоры, но Redux настаивает, что единственный способ обновить данные - это генерация и распространение действий (dispatch actions). Следовательно, Redux и Baobab стремятся решить ту же задачу по-разному и не дополняют друг друга.

В отличии от Immutable, Baobab пока не реализует никаких специальных эффективных структур данных под капотом, так что вы на самом деле ничего не выиграете от использования его вместе с Redux. Проще всего использовать простые объекты в данном случае.

### Rx

[Reactive Extensions](https://github.com/Reactive-Extensions/RxJS) (и ее [современная реализация](https://github.com/ReactiveX/RxJS)) - это превосходный способ управлять сложностью асинхронных приложений. На самом деле [есть попытка создать библиотеку модели взаимодействие человека с компьютером, в качестве взаимозависимых наблюдателей](http://cycle.js.org).


Имеет ли смысл использовать Redux вместе с Rx? Конечно! Они отлично работают вместе. Например, так легко сделать хранилище Redux наблюдаемым:

```js
function toObservable(store) {
  return {
    subscribe({ onNext }) {
      let dispose = store.subscribe(() => onNext(store.getState()));
      onNext(store.getState());
      return { dispose };
    }
  }
}
```

Кроме того, вы можете создавать различные асинхронные потоки, чтобы преобразовать их в действия перед подачей их на `store.dispatch()`.

Вопрос в том, действительно ли вами необходим Redux, если вы уже используете Rx? Возможно нет. Несложно [внедрить Redux в Rx](https://github.com/jas-chen/rx-redux). Некоторые скажут, что это легче простого, используя Rx `.scan()` метод. Это вполне возможно!

Если вы сомневаетесь, изучите исходный код Redux (там не так много изменений), а также его экосистему(например, [developer tools](https://github.com/gaearon/redux-devtools)). Если вы не слишком беспокоитесь об этом и хотите заниматься реактивным программированием, вы, возможно, захотите вместо этого исследовать что-то вроде [Cycle](http://cycle.js.org) или даже вместе с Redux. Дайте нам знать, как у вас дела!