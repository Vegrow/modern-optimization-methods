# Лабораторная работа №4 - Пишем CI/CD файл.

## Цель работы

- познакомиться с плохими и хорошими практиками написания CI/CD файла.

### Напишем "плохой" CI/CD файл
```
name: CI/CD

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build

      - name: Deploy
        run: npm run deploy
```

### Разбираем, что же тут плохо:

1. Процесс CI/CD у нас запускается всегда и только по пушу. Как минимум нужно добавить запуск при создании PR. И, скорее всего, нам будет интересны PR только в main (или dev). А так же, на мой взгляд, нужно убрать запуск по пушу. Мало ли что и зачем пушнул в репу. Давайе экономить ресурсы :)\
**Исправление**: Запускаем скрипт только для пулл-реквестов и только для ветки main. 
 
2. Используется мажорная версия actions (v4). Может привести к непредсказуемому поведению при появлении новых минорых версий.\
**Исправление**: Указать конкретную версию actions.

3. Отсутствует кеширования зависимостей. Каждый раз при выполнении инструкции `npm install` будут заново тянуться все зависимости. Это увеличивает время выполнения. Осуждаем.\
**Исправление**: Использовать кеширование для `node_modules`.

4. Отсутствие предварительной проверки кода на чистоту. Мы же хотим, чтобы в репе было чисто? В этом нам поможет lint!\
**Исправление**: Добавим проверку lint'a перед тестами.

5. Деплой пройдёт даже в том случае, если прошлый шаги будут выполнены с ошибкой. Это нам не надо!\
**Исправление**: Добавим проверку на успешность предыдущих шагов.

6. Отсутствует информирование об ошибках.\
**Исправление**: Добавим информирование. 

### Хороший CI/CD файл

```
name: CI/CD

on: 
  push:
    branches: 
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4.1

      - name: Cache npm dependencies
        uses: actions/cache@v4.1
        with:
          path: ~/.npm
          key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
          restore-keys: ${{ runner.os }}-npm-

      - name: Lint check
        run: npm run lint

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build

      - name: Deploy
        if: success()
        run: npm run deploy
		
      - name: On Failure
        if: failure()
        run: echo "Deployment failed!"
```

## Итоги
В резутате выполняния работы мы познакомились с некоторыми плохими практиками а так же научились их избегать.
