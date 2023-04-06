# Guideline для разработки

Ссылки: [ [Архитектура](https://docs.gemforge.ru/devs/intro) ]

В этом документе собраны основные практики и подходы, которые стоит использовать для разработки клиентской части приложений.

- [Guideline для разработки](#guideline-для-разработки)
  - [Настройки среды](#настройки-среды)
    - [Workspace](#workspace)
    - [Git](#git)
    - [IDE](#ide)
  - [Разработка компонентов React](#разработка-компонентов-react)
    - [Виды компонентов](#виды-компонентов)
    - [Организация файлов](#организация-файлов)
    - [Локализация](#локализация)


## Настройки среды

### Workspace
Основной операционной средой разработки является **Windows 11**, поэтому если возникают какие-то проблемы на другой ОС, желательно обновиться

В качестве командной строки можно использовать, что вам больше нравиться. Неплохим выбором будет:
- [Cmder](https://cmder.app/)
- Git bash

### Git

#### Настройки

В качестве системы контроля версий мы используем **git**. Все основные репозитории хранятся на [**Gemforge GitHub**](https://github.com/Gemforge).
Для управления репозиторием рекомендуется использовать **команндную строку git**

> Не рекомендуем использовать графические клиенты `git`, т.к. даже в официальных приложениях от `GitHub` много скрытых настроек и команд, которые только сбивают с толку и приводят к нежелательным багам

Для соединения с GitHub рекомендуется использовать SSH соединение. Настроить его достаточно просто, нужно лишь сгенерировать пару ключей для рабочей машины и добавить в свой акканут на GitHub. 

[ [Настройка SSH в GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) ]

#### Разработка с git

Для разработки новой функциональности и исправления багов мы используем одинаковый процесс:

- Перед началом новой задачи убедитесь, что вы на `tip of the master` (ваш репозиторий синхронизирован и вы на верху основной ветки). Для этого надо переключиться на `master` и сделать
```
git pull
```
- Создайте новую ветку, указав ваш ник и номер задачи (или смысловое ключевое слово). Например: `evgeny/GEM-120` или `evor/link_block`.
- Отправьте информацию о ветке на `remote` через
```
git push -u origin HEAD
```
- Делайте регулярные и именованные коммиты
```
git commit -m "Добавил новый маршрут для боковой панели Топика"
```
- Если вы работаете несколько дней и есть вероятность, что `master` уже сдвинулся за это время, то стоит использовать `rebase` (в отличии от `merge` подхода - это сократит количество искуственных коммитов)
```
git rebase origin/master
```
- Если у вас есть вопрос по ходу работы или вы просто хотите получить обратную связь - создайте **`Pull Request`**
- `Pull Request` требует, чтобы хотябы один член команды сделал `Approve`
- После этого можете мерджить ваши изменения в `master`

### IDE

Для разработки клиентских приложений на React рекомендуется использовать **VSCode**
Чтобы разработка была более комфортна можно использовать расширения:
- **Prettier - Code formatter**
- Prisma
- GraphQL for VSCode
- EditorConfig for VS Code
- Markdown All in One

## Разработка компонентов React

`React` не описывает жесткие требования к разработке компонентов, и даже официальные рекомендованые подходы меняли со временем.

Поэтому в этом документы мы определимся с тем как делать компоненты, чтобы они были более менее однообразны и было легко переключаться на другие части приложения.

Несколько общих правил:

- Мы используем только **Функциональные Компоненты**
- Мы используем обычные функции, чтобы описывать компоненты на верхнем уровне моделя:
```jsx
// Хорошо:
export function MyComponent() { };

// Хуже
export const MyComponent = () => {};
```

- Мы не стараемся использовать `Typescript` фанатично, но для компонентов стоит описывать типа для их свойств, чтобы не ошибаться и не тратить время, когда компонент надо использовать
```jsx
export type MyComponentProps = {
  name: string;
  age: number;
}

export function MyComponent(props: MyComponentProps) {
  return <div>{props.name} - {props.age}</div>;
}

---
<MyComponent name="Alex" age={37} />
---
```

### Виды компонентов

Можно выделить несколько типов компонентов в приложении:

1. `Simple Components` - простые компоненты, которые не содержат внутреннего состояния. Они принимают набор свойств и рендеряд в зависимости от значений. Нужно стараться делать больше таких компонентов. Их удобно переиспользовать и они не с чем не связаны.
Пример:
```jsx
export function Badge(props: BadgeProps) {
  return (
    <div className={styles.badge}>
      <span>{props.num}</span>
    </div>
  )
}
```


2. `Smart Containers` - компоненты, которые управляют состоянием. Для простых состояний используют `useState`. Такие контейнеры могут передавать состояние другим `простым компонентам` через `props` и принимать изменения через колбэки.

```jsx
export function HitCounter(props: HitCounterProps) {
  const [count, setCount] = useState(0);
  return (
    <div className={styles.counter}>
      <Badge num={count} />
      <Button onClick={() => setCount(count + 1)}>
    </div>
  )
}
```

3. `Data Containers` - компоненты, которые управляют загрузкой данных из вне. Т.к. мы в основном используем `GraphQL`, то загрузка и управление данными обычно происходит через `useQuery` (для запросов) и через `useMutation` (для изменений).
```jsx
export function UserHitCounter(props: UserHitCounterProps) {
  const [data, loading, error] = useQuery(LOAD_USER_HITS);
  const [updateHits] = useMutation(SET_USER_HITS);

  //
  // Помните, что в таких компонентах загрузка выполняется асинхронно, это
  // значит, что функция UserHitCounter будет выполнена несколько раз и вначале data будет `undefined`
  //
  if (!data) {
    return null;
  }

  const { count } = data;
  return (
    <div className={styles.counter}>
      <Badge num={count} />
      <Button onClick={() => updateHits(...)}>
    </div>
  )
}
```

- Желательно, чтобы компоненты делали что-то одно. Загрузка данных отдельно, управление состоянием отдельно и рендеринг какого-то параметризированного участка - отдельно.


### Организация файлов

В наших проектах есть несколько простых правил организации кода:

- Для каждого отдельного компонента желательно использовать отдельный файл, у которого имя файла совпадает с именем компонента:
```
// Хорошо
src/components/MyComponent.tsx:
export function MyComponent {
  ...
}

// Плохо
src/components/MyComponent.tsx:
export function Component {
  ...
}
```
- Если компонент использует стили, которые характерны для него, то желательно использовать `CSS Module` для такого компонента. Название файлы должно совпадать в именем компонента: `MyComponent.module.scss`
- Простые компоненты, которые можно переиспользовать в разных частях приложения храняться в каталоге `components`
- Страницы - это компоненты приложения, которые обычно занимают отдельный путь в приложении. Например путь `/profile` будет соответствовать компоненту `ProfilePage`. Такие компоненты находяться в каталоге `pages`
- `utils` - содержит наборы логически организованных функция для упрощения работы с разными сущностями, как даты, время, HTML, DOM, и так делее.
- `hooks` - содержит наши собственные хуки, которые можно использовать для любых компонентов

### Локализация