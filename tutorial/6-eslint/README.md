# 6 - ESLint

Мы собираемся контролировать качество кода (to lint), чтобы перехватывать потенциальные проблемы. ESLint - наиболее предпочтительный анализатор кода (linter) для ES6. Вместо того чтобы самим определять правила для нашего кода, мы воспользуемся конфигурацией, созданной Airbnb. В этой конфигурации используется несколько плагинов, поэтому мы их тоже установим.

- Запустите `yarn add --dev eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y@2.2.3 eslint-plugin-react`

Как вы видите, вы можете установить несколько пакетов одной командой. Как обычно, они все добавятся в ваш `package.json`.

В `package.json`, добавьте свойство `eslintConfig` со следующим содержанием:

```json
"eslintConfig": {
  "extends": "airbnb",
  "plugins": [
    "import"
  ]
},
```

В разделе `plugins` мы сообщаем ESLint, что используем синтакисис ES6 `import`.

**Примечание**: Вместо свойства `eslintConfig` в `package.json` можно использовать файл `.eslintrc.js`, `.eslintrc.json` или `.eslintrc.yaml` в корне вашего проекта. Так же как и с конфигурацией Babel, мы стараемся избегать загромождения корневой директории большим количеством файлов, но если у вас сложная конфигурация ESLint, рассмотрите такую альтернативу.

Создадим задачу для Gulp, которая будет запускать ESLint. Для этого установим еще и плагин ESLint для Gulp:

- запустите `yarn add --dev gulp-eslint`

Добавьте следующую задачу в ваш `gulpfile.babel.js`:

```javascript
import eslint from 'gulp-eslint';

const paths = {
  allSrcJs: 'src/**/*.js',
  gulpFile: 'gulpfile.babel.js',
  libDir: 'lib',
};

// [...]

gulp.task('lint', () => {
  return gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError());
});
```

Здесь мы сообщаем Gulp, что для этой задачи хотим подключить `gulpfile.babel.js` и JS файлы, расположенные в `src`.

Откорректируйте задачу `build` так, чтобы `lint` предваряла ее запуск:

```javascript
gulp.task('build', ['lint', 'clean'], () => {
  // ...
});
```

- Запустите `yarn start`. Вы должны увидеть набор ошибок кода (linting errors) в этом Gulp-файле и предупреждения об использовании `console.log()` в `index.js`.

Один из видов ошибок будет выглядить следующим образом: `'gulp' should be listed in the project's dependencies, not devDependencies (import/no-extraneous-dependencies)` ('gulp' должен подключаться в разделе `dependencies`, а не `devDependencies`). Вообще-то это неверная ошибка. ESLint не может знать какие JS файлы будут входить только в скомпилированное приложение (build), а какие нет. Поэтому мы немного поможем ESLint, используя комментарии в коде. В `gulpfile.babel.js`, в самом верху, добавьте:

```javascript
/* eslint-disable import/no-extraneous-dependencies */
```

Таким образом, ESLint не будет применять правило `import/no-extraneous-dependencies` в этом файле.

Теперь у нас осталась проблема с `Unexpected block statement surrounding arrow body (arrow-body-style)` (Неожиданное определение блока, окружающего тело стрелочной функции). Это важно. ESLint сообщает нам, что существует лучший способ написать следующий код:

```javascript
() => {
  return 1;
}
```

Это нужно переписать следующим образом:

```javascript
() => 1
```

Потому что, когда ES6 функция содержит только возвращаемое выражение, можно опустить фигурные скобки, оператор return и точку с запятой.

Так что давайте обновим Gulp-файл соответственно:

```javascript
gulp.task('lint', () =>
  gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError())
);

gulp.task('clean', () => del(paths.libDir));

gulp.task('build', ['lint', 'clean'], () =>
  gulp.src(paths.allSrcJs)
    .pipe(babel())
    .pipe(gulp.dest(paths.libDir))
);
```

Последняя оставшаяся проблема связана с `console.log()`. Давайте скажем, что мы хотим в этом примере, чтобы использование `console.log()` в `index.js` было правомерным, а не вызывало предупреждение. Как вы, возможно, догадались, мы поместим `/* eslint-disable no-console */` в начале нашего `index.js` файла.

- Запустите `yarn start` - теперь все снова без ошибок.

**Примечание**: В этой части мы работали с ESLint через консоль. Это хорошо для поиска ошибок во время компиляции / перед публикацией, но вы, так же, возможно, захотите интегрировать его в вашу IDE. НЕ ИСПОЛЬЗУЙТЕ встроенный в вашу среду анализатор кода для ES6. Сконфигурируйте ее так, чтобы для этого использовались модули, расположенные в директории `node_modules`. В этом случае будут использоваться все настройки вашего проекта, правила Airbnb и так далее. Иначе, вы получите лишь усредненный ES6 анализатор.


Следующий раздел: [7 - Клиентское приложение ~на основе Webpack~](/tutorial/7-client-webpack)

Назад в [предыдущий раздел](/tutorial/5-es6-modules-syntax) или [Содержание](/../../#Содержание).
