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
### CSRF - Proteção da rota
Com o helper abaixo o formulário será enviado com um token de segurança
```
<form method="POST" action="/profile">
    {{ csrf_field() }}
</form>
```
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
Você pode dar nomes para as suas rotas, isto é extremamente útil para quando é necessário mudar as urls do site, recomento que sempre seja nomeada.

```
Route::get('user/profile', function () {
})->name('profile');

Route::get('user/profile', 'UserController@showProfile')->name('profile');
```
O nome da rota pode ser como os exemplos abaixo
->route('profile');
->route('profile.show');

**Gerar urls para rotas nomeadas**


```php
//COM PARAMENTROS user/5/profile
// PARA ESTA ROTA
Route::get('user/{id}/profile', function ($id) {
})->name('profile');
$url = route('profile', ['id' => 1]);

// REDIRECIONAR PARA UMA URL NOMEADA
return redirect()->route('profile');
```
**Gerar urls para as rotas nomeadas**
```php
    route('profile');
    route('profile.show');
    route('profile', ['id' => 1]);
```
**Inspecting The Current Route**
Em Breve

# Rotas agrupadas

**Rotas com prefixo**
Você pode criar suas rotas como listado abaixo repetindo o users/ em todas as rotas ou criar um prefixo para agrupar todas as rotas de users evitando assim repetição.

Rotas não agrupadas
```php
    Route::get('users/', 'UsersController@index');
    Route::get('users/show/', 'UsersController@show');
    Route::post('users/create/', 'UsersController@create');
    Route::delete('users/destroy/', 'UsersController@destroy');
```
Rotas agrupadas
```php
Route::prefix('users')->group( function(){
    Route::get('/', 'UsersController@index');
    Route::get('show/', 'UsersController@show');
    Route::post('create/', 'UsersController@create');
    Route::delete('destroy/', 'UsersController@destroy');
});
// ou
Route::group(['prefix' => 'users'], function(){
    Route::get('/', 'UsersController@index');
    Route::get('show/', 'UsersController@show');
    Route::post('create/', 'UsersController@create');
    Route::delete('destroy/', 'UsersController@destroy');
});
```

**Rotas com namespace para o controller**
Por padrão quando uma rota e chamada **Route::get('users/', 'UsersController@index');** o laravel vai procurar o controller defindo na pasta App\Http\Controllers onde foi defindo o namespace padrão do laravel para os controllers: **App\Http\Controllers**

Então caso você queira adicionar namespaces para seus controllers basta seguir o código abaixo, desta forma você poderá colocar seu controller em uma subpasta em App\Http\Controllers\.
Por exemplo você quer que todos os seus controlers do admin estejam dentro de uma pasta /admin

Controller Photo
```php
<?php
namespace App\Http\Controllers\Admin;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PhotoController extends Controller
{
    public function index(){
        echo "Index Photo";
    }
}
```
Rota para os controllers do namespace admin, neste caso somente para o index do PhotoController
```php
<?php
Route::namespace('admin')->group(function () {
    Route::get('admin/photo/', 'PhotoController@index');
});

```

**Subdomínio**
```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```
**Middleware**
Atribui um ou mais middleware para um grupo
```php
Route::middleware('first')->group(function () {
    Route::get('/', 'PhotoController@index');
    Route::get('user/profile', 'PhotoController@create');
});

Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', 'PhotoController@index');
    Route::get('user/profile', 'PhotoController@create');
});
```
# Route Model Binding
Em breve

# Form Method Spoofing
Formulários html não suportam os métodos http, put, patch, e delete. Na verdadde só aceitam post e get, mas as rotas suportam vários métodos

```
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

para enviar um formulário com um método diferente um campo  com o name _method
```
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```
pode-se usar um helper para isto

```
{{ method_field('PUT') }}
```
# Rota atual
Acessa as informações da rota atual

```
$route = Route::current();
$name = Route::currentRouteName();
$action = Route::currentRouteAction();
```

