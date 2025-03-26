### Реализация авторизации пользователей в Yii2


#### Шаг 1: Настройка модели User для авторизации

1. Откройте `models/User.php`
2. Перенесите код из ранее переименованной модели (например, `UserOld`)
3. Добавьте к классу User `implements IdentityInterface`

```php
use yii\web\IdentityInterface;

class User extends \yii\db\ActiveRecord implements IdentityInterface
{
    // ... существующий код ...

    public static function findIdentity($id)
    {
        return static::findOne($id);
    }

    public static function findIdentityByAccessToken($token, $type = null)
    {
        return null; // Не используем токены
    }

    public function getId()
    {
        return $this->id;
    }

    public function getAuthKey()
    {
        return null; // Не используем authKey
    }

    public function validateAuthKey($authKey)
    {
        return false; // Не используем authKey
    }

    public static function findByUsername($username)
    {
        return static::findOne(['username' => $username]);
    }

    public function validatePassword($password)
    {
        return Yii::$app->security->validatePassword($password, $this->password);
    }
}
```


---


#### Шаг 2: Изменение формы входа

1. Изменяем `models/LoginForm.php` (добавляем 'message' => 'Заполните поле' для обязательных полей):

```php

    public function rules()
    {
        return [
            [['username', 'password'], 'required', 'message' => 'Заполните поле'],   
            ['rememberMe', 'boolean'],
            ['password', 'validatePassword'],
        ];
    }


```


---

#### Шаг 3: Изменяем представления для входа

1. Создаем `views/site/login.php`:

```php
<?php


$this->title = 'Авторизация';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-login">
    <h1><?= Html::encode($this->title) ?></h1>

    <p>Пожалуйста заполните все поля:</p>

    <div class="row">
        <div class="col-lg-5">

            <?php $form = ActiveForm::begin([
                'id' => 'login-form',
                'fieldConfig' => [
                    'template' => "{label}\n{input}\n{error}",
                    'labelOptions' => ['class' => 'col-lg-1 col-form-label mr-lg-3'],
                    'inputOptions' => ['class' => 'col-lg-3 form-control'],
                    'errorOptions' => ['class' => 'col-lg-7 invalid-feedback'],
                ],
            ]); ?>

            <?= $form->field($model, 'username')->textInput(['autofocus' => true])->label('Имя пользователя') ?>

            <?= $form->field($model, 'password')->passwordInput()->label('Пароль') ?>

            <?= $form->field($model, 'rememberMe')->checkbox([
                'template' => "<div class=\"custom-control custom-checkbox\">{input} {label}</div>\n<div class=\"col-lg-8\">{error}</div>",
            ])->label('Запомнить меня') ?>

            <div class="form-group">
                <div>
                    <?= Html::submitButton('Войти', ['class' => 'btn btn-primary', 'name' => 'login-button']) ?>
                </div>
            </div>

            <?php ActiveForm::end(); ?>

            <div style="color:#999;">
                Уважаемые члены комиссии, данные авторизации: <br> <strong>admin/admin</strong> или <strong>demo/demo</strong>.<br>
    
            </div>

        </div>
    </div>
</div>

```

![image](https://github.com/user-attachments/assets/8bd73f8c-f1fa-4938-8a08-ccf639db094b)


---

#### Шаг 4: Проверка авторизации в других контроллерах

Для ограничения доступа к определенным действиям используем behaviors:

```php
public function behaviors()
{
    return [
        'access' => [
            'class' => \yii\filters\AccessControl::className(),
            'rules' => [
                [
                    'allow' => true,
                    'actions' => ['login', 'register'],
                    'roles' => ['?'], // Только для гостей
                ],
                [
                    'allow' => true,
                    'roles' => ['@'], // Только для авторизованных
                ],
            ],
        ],
    ];
}
```

*картинка с (настройка доступа в контроллере)*

---

#### Шаг 7: Проверка работы авторизации

1. Введите данные зарегистрированного пользователя
2. Проверьте:
   - После успешного входа происходит перенаправление
   - В верхнем меню отображается имя пользователя
   - Кнопка "Выйти" работает корректно


![image](https://github.com/user-attachments/assets/4fe08fa3-a875-4245-8d38-fd67afa34ae3)


---

### Итог

Мы реализовали:
1. Адаптированную модель User для авторизации
2. Форму входа с валидацией
3. Контроллер для обработки входа/выхода


Теперь система имеет полноценную авторизацию, интегрированную с нашей базой данных.
