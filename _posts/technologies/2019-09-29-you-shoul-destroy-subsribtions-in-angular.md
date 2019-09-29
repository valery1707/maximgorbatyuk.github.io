---
layout: post
title: Не забудьте отписаться. Subject Subscribtions в Angular и RxJS
category: technologies
tags: [development, technologies, angular, rxjs]
---

Источник: [https://brianflove.com](https://brianflove.com/2016/12/11/anguar-2-unsubscribe-observables/)

---

# Зачем
Когда ты реализуешь свой компонент с участием подписок на Subject, то ты должен написать и отмену подписки, иначе будут утечеки памяти.

Даже если зайти в описание метода [ngOnDestroy](https://angular.io/guide/lifecycle-hooks#lifecycle-sequence), то мы увидим рекомендацию для этого:

> `ngOnDestroy()`: Cleanup just before Angular destroys the directive/component. Unsubscribe observables and detach event handlers to avoid memory leaks.

В общем, ответ на вопрос "Когда нам лучше всего отписываться" у нас есть: мы пишем код отмены подписок в реализацию метода ngOnDestroy, чтобы она была запущена перед уничтожением компонента.

# Как отменить подписки

Для начала определим код компонента и подписки:

```typescript

export class MyComponent implements OnDestroy {

    private subscription: ISubscription;

    constructor() {
        // awesome initialization goes here
    }

    ngOnDestroy() {
        // awesome code goes here
    }

    this.counter = new Observable<number>(observer => {
        console.log('Subscribed');
        let index = -1;
        const interval = setInterval(() => {
            index++;
            console.log(`next: ${index}`);
            observer.next(index);
        }, 1000);

        // teardown
        return () => {
            console.log('Teardown');
            clearInterval(interval);
    }

}

```

Есть три способа:

## 1. unsubscribe()

Нужно сохранить подписку в компоненте и вызвать метод unsubscribe()

```typescript

ngOnInit(): {
    const subscription = this.counter.subscribe(value => console.log(value));
    this.subscription.add(subscription);
}

ngOnDestroy() {
    this.subscription.unsubscribe();
}

```

## 2. takeWhile()

Можно передать в этот метод некое булево свойство, и пока она `true`, подписка будет сохранена в памяти.

```typescript

this.alive = true;

this.counter
    .pipe(takeWhile(() => this.alive))
    .subscribe(
      (value) => this.count = value,
      (error) => console.error(error),
      () => console.log('[takeWhile] complete')
    );

ngOnDestroy() {
    this.alive = false;
}

```

С этим оператором есть определенные проблема - вызов `takeWhile()` и проверка необходимости отписки происходит непосредственно сторонним событием. Получается, что если подписка отправляет некоторые данные "раз в час", то после уничтожения компонента есть вероятно, что эта подписка "провисит" в памяти этот самый час, пока не будет вызвана проверка условия отмены подписки.

## 3. takeUntil()

`takeUntil()` более предпочтителен, так как отмена подписки базируется на другой подписке, которой управляем уже мы. Мы создаем специальный `subject` в нашем компоненте и передаем его в серверную подписку как триггер отписки.

```typescript

private unsubscribe: Subject<void> = new Subject();

this.counter
    .pipe(takeUntil(this.unsubscribe))
    .subscribe(
      (value) => this.count = value,
      (error) => console.error(error),
      () => console.log('[takeWhile] complete')
    );

ngOnDestroy() {
    this.unsubscribe.next();
    this.unsubscribe.complete();
}

```