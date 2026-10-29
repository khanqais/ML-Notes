## EventEmitter `.on()` — Short Note

- `eventEmitter.on('start', callback)` **registers a listener** for the event named `'start'`.
- The callback does **not run immediately**; it runs only when the event is emitted.
- `eventEmitter.emit('start')` triggers the event and executes all listeners attached to `'start'`.

```js
eventEmitter.on('start', () => {
  console.log('Useless event');
});

eventEmitter.emit('start'); // prints: Useless event
```

- `on()` = **listen/subscribe**
- `emit()` = **trigger/fire**
- Used for event-driven, loosely coupled code (e.g., `userRegistered`, `orderPlaced`).