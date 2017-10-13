### Migrations
Controle de versão e gerenciamento da estrutura das tabelas
É uma ou várias classes que ao serem executadas exetutam operações no banco de dados
Os arquivos são gerados na pasta database/migrations da sua aplicação.
***Criar uma Classe migration***
Básicamente são 3 comandos


Cria uma migração vasia, método up e down vasios
***php artisan make:migration cars***
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class Cars extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        //
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```


Cria uma migração preparada para criar uma tabela, método up e down já com código para gerar a tabela com campos básicos
***php artisan make:migration create_cars_table --create=cars***
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCarsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('cars', function (Blueprint $table) {
            $table->increments('id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('cars');
    }
}

```

Cria uma migração para adicionar/remover campos e alterar a tabela, método up e down já com código para alterar a tabela
***php artisan make:migration add_ano_table --table=cars***
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AddAnoTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('cars', function (Blueprint $table) {
            //
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('cars', function (Blueprint $table) {
            //
        });
    }

```

Comando alterantivo para criar migrations para as tabelas
***php artisan make:migration create_produtos_table***
Vai gerar uma migration para a tabela produtos o arquivo encontra-se na pasta ***database/migrations*** da sua aplicação.
veja o arquivo gerado

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateProdutosTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('produtos', function (Blueprint $table) {
            $table->increments('id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('produtos');
    }
}
```

Esta classe possui dois métodos up e down o up é para criar ou atualizar, o down é para desfazer, ambos você tem que programar o que vai ser realizado.

para rodar a migrations métodos up
***php artisan migrate***

Vai rodar a última migração 
***php artisan migrate:rollback***

Rollback em toda a aplicação
***php artisan migrate:reset***

Rollback em toda a aplicação e executa a migration
***php artisan migrate:refresh***

Rollback em toda a aplicação e executa a seed
***php artisan migrate:refresh --seed***

### Criar as tabelas
Pode-se criar a tabela e depois alterar adicionar campos
Por padrão o laravel coloca dois campos quando cria a migration, dentro do método up:

cada linha representa um campo, com algumas execessões como no caso do ***$table->timestamps()*** que vai criar 2 campos created_at e update_at.
```php
    $table->increments('id');
    $table->timestamps();
```
rode a migração
***php artisan migrate***

Olhe a tabela, o laravel criou a tabela e colocou os campos lá id, created_at, e update_at.

### Criar/Adicionar uma nova coluna
Crie uma nova migração
***php artisan make:migration add_column_produtos --table=produtos***

Código gerado 
```php

<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AddColumnProdutos extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('produtos', function (Blueprint $table) {
            // SEUS CAMPOS AQUI
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('produtos', function (Blueprint $table) {
            //
        });
    }
}
```
***Adicionando novos campos***
No método up:
```php
    Schema::table('produtos', function (Blueprint $table) {
        // SEUS CAMPOS AQUI
        $table->string('nome', 100); // Varchar
        $table->float('preco', 8, 2); // FLOAT
        $table->boolean('ativo'); ///
    });
```

Você pode encontrar mais campos aqui:
https://laravel.com/docs/5.5/migrations

Rode o comando ***php artisan migrate*** e os campos serão adicionados a tabela produto.

***Modificar a tabela***
Para isto o laravel depende de um pacote do doctrine então instale com o composer:
***composer require doctrine/dbal***

