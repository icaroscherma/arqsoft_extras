## CakePHP 3 Testing
O conteúdo abaixo foi baseado no [link](http://book.cakephp.org/3.0/en/development/testing.html).

Inicialmente após se instalar o CakePHP 3 é necessário adicionar o ``"phpunit/phpunit": "*",`` dentro do campo ``require`` do ``/composer.json``, e depois rodar o comando ``composer install`` (para instalar novos pacotes) ou ``composer update`` (atualizar todos os pacotes e instalar os novos).

Feito isso, tenho como escopo o _Controller_ abaixo:

``projeto/src/Controller/ValuesController.php``
```PHP
<?php
namespace App\Controller;
use App\Controller\AppController;

class ValuesController extends AppController
{
    // Runs when the object is constructed
    public function initialize()
    {
        // Also inherit the initialize() from the parent (AppController)
        parent::initialize();

        // Enable the use of the RequestHandler (JSON, XML...)
        $this->loadComponent('RequestHandler');
    }

    public function index()
    {
        // Force the page to be rendered as a JSON response
        $this->RequestHandler->renderAs($this, 'json');

        // Create a var with an value
        $pi = '3.14';

        // Enable the "$pi" to be used outside the controller. ($this->set)
        // "compact" function accepts an array of vars without the need of "$"
        $this->set(compact('pi'));

        // Get the var already outside the controller scope and serialize it.
        $this->set('_serialize', ['pi']);
    }
    

    // Same thing of index() but with a different value
    public function e()
    {
        $this->RequestHandler->renderAs($this, 'json');
        $e = '2.71828';
        $this->set([
            '_serialize' => ['e'],
            'e' => $e
        ]);
    }

}
```

Onde basicamente se entrar por exemplo ``http://localhost/values/`` terá como retorno o ``JSON`` com o valor de ``PI``, e caso entre em ``http://localhost/values/e`` terá como retorno um ``JSON`` com o valor de ``E`` (função matemática).

Agora vamos para a parte do teste:

``projeto/tests/TestCase/Controller/ValuesControllerTest.php``

```PHP
<?php
namespace App\Test\TestCase\Controller;

use Cake\TestSuite\IntegrationTestCase;

class ValuesControllerTest extends IntegrationTestCase
{
    public function testPi()
    {
        // Tells that the header must be JSON
        $this->configRequest([
            'headers' => ['Accept' => 'application/json']
        ]);

        // Get the contents from "ApplicationRoot->ValuesController"
        $result = $this->get('/values/');

        // Check that the response was a 200
        $this->assertResponseOk();

        // Tells you the expected content
        $expected = [
            'pi' => '3.14',
        ];

        // Encodes from PHP array to JSON
        $expected = json_encode($expected, JSON_PRETTY_PRINT);

        // Get the result if the assert $expected is equal to the response (body content)
        $this->assertEquals($expected, $this->_response->body());
    }

    public function testE()
    {
        // Tells that the header must be JSON
        $this->configRequest([
            'headers' => ['Accept' => 'application/json']
        ]);

        // Get the contents from "ApplicationRoot->ValuesController->E_Function"
        $result = $this->get('/values/e');

        // Check that the response was a 200
        $this->assertResponseOk();

        // Tells you the expected content
        $expected = [
            'e' => '2.71828',
        ];

        // Encodes from PHP array to JSON
        $expected = json_encode($expected, JSON_PRETTY_PRINT);

        // Get the result if the assert $expected is equal to the response (body content)
        $this->assertEquals($expected, $this->_response->body());
    }
}
```

Pronto, com isso você deve ser capaz de rodar um teste simples, sem uso de banco de dados, fixture ou outras firulas.
Para rodar, no terminal execute ``/vendor/bin/phpunit "nome-do-teste"``, no caso: ``/vendor/bin/phpunit tests/TestCase/Controller/ValuesControllerTest.php``.
E o resultado experado deve ser:
```
PHPUnit 4.6.8 by Sebastian Bergmann and contributors.

Configuration read from /Applications/XAMPP/xamppfiles/htdocs/projects/arqsoft_ws/phpunit.xml.dist

..

Time: 88 ms, Memory: 8.25Mb

OK (2 tests, 10 assertions)
```

Obs.: ``arqsoft_ws`` é a raiz do meu projeto na minha maquina.

No mais é isso.
