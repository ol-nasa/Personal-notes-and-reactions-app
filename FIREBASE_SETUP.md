# Настройка Firebase для синхронизации данных

Это приложение поддерживает облачную синхронизацию данных через Firebase. Следуйте этим шагам для её включения.

## Шаг 1: Создайте Firebase проект

1. Откройте [Firebase Console](https://console.firebase.google.com/)
2. Нажмите **"Создать проект"**
3. Введите имя проекта (например: `notes-app`)
4. Отключите Google Analytics (не нужна для нашего приложения)
5. Нажмите **"Создать"** и ждите ~1 минуту

## Шаг 2: Создайте веб-приложение

1. В Firebase Console нажмите иконку **`</>`** (Add app)
2. Выберите **"Web"**
3. Введите имя приложения (например: `notes-web`)
4. Нажмите **"Register app"**
5. Скопируйте конфиг (увидите объект вида):
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyA...",
  authDomain: "notes-app-xxx.firebaseapp.com",
  projectId: "notes-app-xxx",
  storageBucket: "notes-app-xxx.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc..."
};
```

## Шаг 3: Включите Firestore

1. В левой панели выберите **"Build"** → **"Firestore Database"**
2. Нажмите **"Create database"**
3. Выберите **"Start in test mode"** (безопасность настроим позже)
4. Выберите регион (например, `europe-west1`)
5. Нажмите **"Create"**

## Шаг 4: Настройте безопасность

1. Откройте **Firestore** → вкладка **"Rules"**
2. Замените правила на:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/notes/{document=**} {
      allow read, write: if request.auth.uid == userId;
    }
  }
}
```

3. Нажмите **"Publish"**

## Шаг 5: Вставьте конфиг в приложение

1. Откройте файл `index.html` в текстовом редакторе
2. Найдите секцию:
```javascript
window.firebaseConfig = {
  apiKey: "AIzaSyA-example",
  authDomain: "notes-app-example.firebaseapp.com",
  projectId: "notes-app-example",
  storageBucket: "notes-app-example.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};
```

3. Замените все значения на скопированные из Firebase Console
4. Сохраните файл и закоммитьте:
```bash
git add index.html
git commit -m "Add Firebase config"
git push origin main
```

## Готово!

Теперь приложение будет:
- 🔄 **Синхронизировать** данные между устройствами автоматически
- 💾 **Сохранять** записи в облаке (Firebase Firestore)
- 📱 **Загружать** последние данные при открытии на любом устройстве

### Как это работает:

1. **На устройстве A** - добавляю запись
2. **Автоматически** - данные сохраняются в Firebase
3. **На устройстве B** - через ~2 секунды появляется запись

### Проверка:

1. Откройте приложение на двух устройствах
2. На первом устройстве добавьте запись
3. На втором устройстве должна появиться в реальном времени
4. Откройте консоль браузера (F12) - увидите "✓ Firebase инициализирован"

### Если что-то не работает:

1. Откройте консоль браузера (F12)
2. Посмотрите сообщения об ошибках
3. Проверьте:
   - ✓ Конфиг Firebase вставлен правильно
   - ✓ Firestore Database создана
   - ✓ Правила безопасности опубликованы

### Отключение Firebase (если нужно):

Если Firebase недоступен или вы хотите использовать только localStorage:
- Оставьте конфиг с `example` в названиях
- Приложение будет работать в режиме локального хранилища (без синхронизации)

---

**Нужна помощь?** Проверьте консоль браузера (F12 → Console) - там будут детальные сообщения об ошибках.
