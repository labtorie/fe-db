## Основные компоненты

### 1. Call Stack (Стек вызовов)

- Хранит контексты выполнения функций
- Работает по принципу LIFO (Last In, First Out)
- Синхронный код выполняется здесь

### 2. Web APIs

- Предоставляются браузером
- `setTimeout`, `setInterval`, `fetch`, DOM events, `requestAnimationFrame`
- Обрабатывают асинхронные операции

### 3. Task Queue (Macrotask Queue)

- Очередь макрозадач
- Содержит: `setTimeout`, `setInterval`, I/O операции
- Обрабатывается после очистки Call Stack

### 4. Microtask Queue

- Очередь микрозадач
- Содержит: `Promise.then/catch/finally`, `queueMicrotask`, `MutationObserver`
- **Приоритет выше, чем у макрозадач**

## Как работает Event Loop

```
1. Выполнить весь синхронный код (Call Stack)
2. Выполнить ВСЕ микрозадачи (пока очередь не пуста)
3. Отрендерить изменения (если нужно)
4. Взять ОДНУ макрозадачу из Task Queue
5. Вернуться к шагу 1
```

## Порядок выполнения

```javascript
console.log('1'); // Call Stack

setTimeout(() => console.log('2'), 0); // Macrotask

Promise.resolve().then(() => console.log('3')); // Microtask

console.log('4'); // Call Stack

// Вывод: 1, 4, 3, 2
```

## Приоритеты задач

**Высший → Низший:**

1. **Синхронный код** (Call Stack)
2. **Microtasks** (Promises, queueMicrotask)
3. **Rendering** (при необходимости)
4. **Macrotasks** (setTimeout, setInterval)

## Важные особенности

### Microtasks выполняются полностью

```javascript
Promise.resolve().then(() => {
  console.log('1');
  Promise.resolve().then(() => console.log('2'));
});
Promise.resolve().then(() => console.log('3'));

// Вывод: 1, 3, 2
// Все микрозадачи до следующей макрозадачи
```

### Бесконечный цикл микрозадач блокирует рендеринг

```javascript
function loop() {
  Promise.resolve().then(loop); // ⚠️ Блокирует UI
}
loop();
```

### setTimeout(fn, 0) не значит "немедленно"

- Минимальная задержка ~4ms (в зависимости от браузера)
- Выполнится после всех микрозадач

## Специальные случаи

### requestAnimationFrame

- Выполняется **перед рендерингом**
- Не macrotask и не microtask
- Идеален для анимаций (~60 FPS)

### requestIdleCallback

- Выполняется когда браузер **простаивает**
- Самый низкий приоритет

## Практический пример

```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout 1');
  Promise.resolve().then(() => console.log('promise in timeout'));
}, 0);

Promise.resolve()
  .then(() => {
    console.log('promise 1');
    setTimeout(() => console.log('timeout in promise'), 0);
  })
  .then(() => console.log('promise 2'));

console.log('end');

/* Результат:
start
end
promise 1
promise 2
timeout 1
promise in timeout
timeout in promise
*/
```

## Типичные ошибки

❌ **Думать что setTimeout(fn, 0) выполнится сразу**

```javascript
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
// Сначала promise, потом timeout
```

❌ **Не учитывать очередь микрозадач**

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log('timeout'));
  Promise.resolve().then(() => console.log('promise'));
}
// promise x3, затем timeout x3
```

## Полезные инструменты

- **Chrome DevTools** → Performance tab
- [Loupe](http://latentflip.com/loupe/) - визуализация Event Loop
- `performance.mark()` и `performance.measure()` для профилирования