# Middlewares no laravel 5.5
Middlewares no Laravel funcionam como filtros para as requisições http.
Exemplo: verificar se o usuário es tá logado, se sim continua a execução, senão redireciona para o login.
O laravel possui alguns middlewares por padrão, AUTH,API,CORS,CSRF.

**Criar um middleware laravel**
Verificar a idade do usuário.

Usando o artisan: **php artisan make:middleware CheckAge**
Este comando vai criar uma classe no diretório **app/Http/Middleware**

```
// SEM A LÓGICA DE VERIFICAÇÃO
namespace App\Http\Middleware;
use Closure;
class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return $next($request);
    }
}
// COM A LÓGICA DE VERIFICAÇÃO
namespace App\Http\Middleware;
use Closure;
class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }
        return $next($request);
    }
}
```
Se o parãmetro age for menor ou igual a 200 ele redireciona para a home senão ele passa o request adiante, isto é continua a execução. 
**return $next($request);**

Depois de criar o middleware é necessário registrar o mesmo, para isso acesse o arquivo /app/Http/Kernel.php, procure o array $routeMiddleware( middleware de rotas ) e adicione o caminho para o arquivo.


```php
// kernel classe
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'check.age' => \App\Http\Middleware\CheckAge::class, // -- verificador de idade -- //
];
```

para usar o middleware
```php
Route::get('middleware', 'FiltroController@index')->middleware('check.age');
```
Para usar vários middlewares
```php
Route::get('middleware', 'FiltroController@index')->middleware(['check.age','auth']);
```
Em um grupo de rotas e com prefixo
```php
Route::middleware('check.age')->prefix('middleware')->group(function(){
    Route::get('/', 'FiltroController@index');
    Route::get('/list', 'FiltroController@list');
});
```

# Middleware Agrupado
Em algumas situações pode ser necessário utilizar vários middlewares, vamos supor que sejam idade e nome, já temos o verificador de idade  vamos criar o de nome
**php artisan make:middleware CheckName**
Não esqueça de registrar.
```php
namespace App\Http\Middleware;
use Closure;
class CheckName
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        // verifica se o parâmetro nome é thiago
        if ($request->name != 'thiago') {
            return redirect('/');
        }
        return $next($request);
    }
}
```

Podemos usar assim
```php
Route::middleware(['check.age','check.name'])->prefix('filtro')->group(function(){
    Route::get('/', 'FiltroController@index');
    Route::get('/list', 'FiltroController@list');
});
```
Ou podemos criar um grupo de middleware onde os dois serão executados. 

No arquivo /app/Http/Kernel.php procure o array $middlewareGroups e adicione o código abaixo, assim estamos criando um middleware que utiliza dois middlewares
```php
    'pessoa' => [
        'check.age' => \App\Http\Middleware\CheckAge::class, // -- verificador de idade -- //
        'check.name' => \App\Http\Middleware\CheckName::class, // -- verificador de nome -- //
    ],
```

Para usar o grupo 
```php
Route::middleware('pessoa')->prefix('filtro')->group(function(){
    Route::get('/', 'FiltroController@index');
    Route::get('/list', 'FiltroController@list');
});
```

# Passando parâmetros para o middleware
caso você necessite passar parâmetros para os seus middlewares você pode, vamos pensar que o middleware checkAge em alguns casos pode  verificar se a idade mínima é de 21 anos e em outros pode verificar se a idade é de 18.
Para isto eu preciso informar ao middleware a idade que deverá se comparada com o parâmetro de url age ?age=30.

```php
namespace App\Http\Middleware;
use Closure;
class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next, $idade = 200)
    {
        if ($request->age <= $idade) {
            return redirect('/');
        }
        return $next($request);
    }
}
```

Repare que a assinatura do método handle agora recebe um parâmetro idade que por padrão é 200 e o método vai comparas a variável idade com o parâmetro vindo do request age.
Veja o código abaixo
```php
Route::middleware(['check.age:18','check.name'])->prefix('middleware')->group(function(){
    Route::get('/', 'FiltroController@index');
    Route::get('/list', 'FiltroController@list');
});
```
Da forma com fizemos acima passamos um parâmetro: 'check.age:18'
Para passar mais parâmtros é só separar por vírgula: 'check.age:18,20,30'
No método handle do seu middleware adicione os paramentros e pronto. handle($request, Closure $next, $idade = 200, **$p2, $p3**)

# Middleware global
É usado para todas as requisições, na verdade você não cria um middleware global você cria um middleware e adiciona ele globalmente, para que em toda requisição ele seja chamado.

Para isto é necessário apenas registrá-lo no vetor $middleware do arquivo  /app/Http/Kernel.php então ele será chamado em todos os requests.
No exemplo abaixo eu adicionei o middleware na última linha
```php
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
        \App\Http\Middleware\CheckName::class, // -- verificador de nome -- //
    ];
```

O exemplo acima vai causar um loop infinito porque internamente ele possui um redirecionamento para a home porque o redirecionamento não tem o parâmentro name=thiago, mas o exemplo foi somente para mostrar como colocar a classes no vetor.


# Before & After Middleware
Isto quer dizer: executar alguma coisa antes ou depois de executar a função anonima $next no método handle do middleware

```php
class BeforeMiddleware // <----- NOME DO SEU MIDDLEWARE
{
    public function handle($request, Closure $next)
    {
        // SEU CÓDIGO AQUI
        return $next($request);
    }
}

class AfterMiddleware // <----- NOME DO SEU MIDDLEWARE
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);
        // SEU CÓDIGO AQUI
        return $response;
    }
}
```
Percebeu?

Before == Antes
Executa seu código antes do next no método handle do middelware
```php
// SEU CÓDIGO AQUI
return $next($request);
```


After == Depois
Executa seu código depois do next no método handle do middelware
```php
$response = $next($request);
// SEU CÓDIGO AQUI
return $response;
```
# Terminable Middleware
Executa alguma operação depois de executar middleware, mas antes do enviar a requisição para o navegador.
Esta operação deve estar no método terminate, que será executado após executar o middeware e antes de enviar a requisição para o navegador.

```php
namespace App\Http\Middleware;
use Closure;
class CheckName
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {

        if ($request->name != 'thiago') {
            return redirect('/?name=thiago');
        }
        return $next($request);
    }
    public function terminate($request, $response)
    {
        // Store the session data...
    }
}
```

