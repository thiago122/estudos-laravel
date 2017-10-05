# Rotas no laravel 5.5

### Básico

 ```
 Route::method($uri, $callback);
  ```

 **Method**: Método http que pode ser: get, post, put, patch, delete, options
 ```php
 Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
 ```
 Usada para mais de um método http usa-se o **match**
 ```
 Route::match(['get', 'post'], '/', function () {
    //
});
 ```
  Usada para todos os métodos http usa-se o **any**
 ```
Route::any('foo', function () {
    //
});
 ```


 **Uri**: caminho para o recurso do seu site(endereço)
 Exemplos de uri:
 /
 /carros
 /carros/5
 /carros/minha-pagin-de-carro

 **Callback**: instrução a ser executada

Rota executando uma função anônima
```php
Route::get('foo', function () {
    return 'Hello World';
});
```
Rota direcionando para um controller
```php
Route::get('/user', 'UsersController@index');
```
### CSRF Protection
Em breve
### Redirecionameto
```php
Route::redirect('/daqui', '/para_ca','redirect_code');
Route::redirect('/usuario', '/users','301');
```

### View Routes
Chama a view sem a necessidade de executar um controller ou uma função anônima
No exemplo abaixo quando for a uri **/welcome** chamara a view **view_welcome**
```php
Route::view('/welcome', 'view_welcome');
```
Passando Dados para a view através de um vetor
No exemplo abaixo: chave **name** valor **Taylor**

```php
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

### Recebendo parametros
Recebendo parâmetros de url
no exemplo abaixo a rota aguarda **uri/parametro** - **user/5** ou **user/id-string**

```php
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```
Quando na Uri utilizamos assim **{id}** o parâmentro é obrigatório,
Para que não seja obrigatório adicione uma interrogação **{id?}**

Vários parâmetros
```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
});
```

### Regular Expression Constraints - Não Resumido
You may constrain the format of your route parameters using the where method on a route instance. The where method accepts the name of the parameter and a regular expression defining how the parameter should be constrained:

```php
Route::get('user/{name}', function ($name) {
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```
### Global Constraints - Não Resumido
If you would like a route parameter to always be constrained by a given regular expression, you may use the pattern method. You should define these patterns in the boot method of your  RouteServiceProvider:
```php
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```
Once the pattern has been defined, it is automatically applied to all routes using that parameter name:
```php
Route::get('user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

## Rotas nomeadas
Você pode dar nomes para as suas rotas, isto é extremamente útil para quando é necessário mudar as urls do site recomento que sempre seja nomeada

```
Route::get('user/profile', function () {
})->name('profile');

Route::get('user/profile', 'UserController@showProfile')->name('profile');
```





