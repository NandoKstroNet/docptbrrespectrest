Documentação PT-BR Respect/Rest
==================
http://nandokstronet.github.io/respect-rest-docs-br/

Tradução da documentação do Respect/Rest PT-BR
===================
<script type="text/javascript" src="http://www.ohloh.net/p/604580/widgets/project_factoids_stats.js"></script>
Respect\Rest [![Build Status](https://secure.travis-ci.org/Respect/Rest.png)](http://travis-ci.org/Respect/Rest)
============

Controlador elegante para aplicações Restful e criação de APIs..

 * Muito leve e elegante.
 * Não faz mudanças em seu PHP, pequena curva de aprendizado.
 * Completamente RestFul, caminho certo para construir aplicações.


Instalação
------------

Packages podem ser encontrados no [PEAR](http://respect.li/pear) e [Composer](http://packagist.org/packages/Respect/Rest). Autoloading é compativél com [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md).

Feature Guide
-------------
### Navigation
[Configuration][] | [Dispatching][] | [Simple Routing][] | [Parameters][] | [Catch-all][] | [Matching][] | [Methods][] | [Controllers][] | [Streams][] | [Static][] | [Forwarding][] | [When][] | [By][] | [Through][] | [Controller Splitting][] | [Conneg][] | [Basic Auth][] | [User-Agent][] | [Content-Type][] | [HTTP Errors][] | [RESTful Extras][] | [Anti-Patterns][] | [Own Routines][] | [Error Handling][]

### Configuration
[Top][]

Bootstrap é bem simples, basta criar uma instância de Respect\Rest\Router.
```php
    use Respect\Rest\Router;

    $r3 = new Router;
```

Pressupomos que você tenha um .htaccess que redireciona para este arquivo PHP e que esteja executando-o a partir da raiz do dominio (http://example.com/ sem qualquer subpasta).

Se você deseja utiliza-lo a partir de uma subpasta, basta passar a caminho para o router:
```php
    $r3 = new Router('/myapp');
```

Com isto o roteador irá trabalhar a partir de: http://example.com/myapp/

Você também pode utilizar o Router sem utilização de um aquivo .htaccess, desta maneira você usará variável CGI PATH_INFO, você pode declara-lo da seguinte maneira:
```php
    $r3 = new Router('/index.php/');
```

The same goes for folders:
```php
    $r3 = new Router('/myapp/index.php/');
```

Isto presupões que as URLs no projeto iniciarão com esses namespaces.

### Dispatching
[Top][]

O router é auto-executado, você não precisa realizar nenhuma ação para que o mesmo execute de fato, a não ser definir as rotas. Mas caso queira omitir tal comportamento proceda da seguinte maneira:
```php
    $r3->isAutoDispatched = false;
```

Note que para ver as excessões é preciso seguir o próximo passo.

You can then dispatch it yourself at the end of the proccess:
```php
    print $r3->run();
```

Você pode imprimir a saída ou armazena-la em uma variável, se assim desejar. Isso permite um melhor teste e a integração do Router com sua app existente.

### Simple Routing
[Top][]

Um "Olá Mundo" ficaria assim:
```php
    $r3->get('/', function() {
        return 'Hello World';
    });
```

Ao acessar http://localhost (Considere sua configuração local para isto) a saída em seu browser será "Hello World". Você pode declarar como rotas o que você desejar:
```php
    $r3->get('/hello', function() {
        return 'Hello from Path';
    });
```

Neste caso, ao acessar http://localhost/hello a saída em seu browser será "Hello from Path".

### Using Parameters
[Top][]

Você pode declarar rotas que recebam parâmetros da URL.Para isso cada parâmetro é um /* no caminho da rota. Considerando o exemplo anterior:
```php
    $r3->get('/users/*', function($screenName) {
        echo "User {$screenName}";
    });
```

Acessando http://localhost/users/alganet ou qualquer outro nome de usuário, além do Alganet, o resultado será "User alganet"(ou o nome escolhido por você.).Multiplos parâmetros podem ser definidos:

```php
    $r3->get('/users/*/lists/*', function($user, $list) {
        return "List {$list} from user {$user}.";
    });
```

Os últimos parâmetros na rota são opcionais por default, basta declarar apenas um `->get('/posts/*'` que irá corresponder a `http://localhost/posts/` sem nenhum parâmetro. Você pode declarar um segundo `->get('/posts'`, agora o Router irá corresponder corretamente, ou tratar o parâmetro ausente, tornando-os anulavéis na função passada.
```php
    $r3->get('/posts/*/*/*', function($year,$month=null,$day=null) {
        /** list posts, month and day are optional */
    });
```

 1. O código acima irá corresponder a /posts/2010/10/10, /posts/2011/01 e /posts/2010.
 2. Os parametros opcionais são aceitos apenas ao fim do path. A seguinte forma não aceita parametro opcional: `/posts/*/*/*/comments/*`

### Catch-all Parameters
[Top][]

Existem casos em que você precisa pegar um número indefinido de parâmetros. Você pode usar o Route com o parametro catch-all, veja:
```php
    $r3->get('/users/*/documents/**', function($user, $documentPath) {
        return readfile(PATH_STORAGE. implode('/', $documentPath));
    });
```

 1. O exemplo citado acima corresponde a `/users/alganet/documents/foo/bar/baz/anything`.
  Como retorno o parâmetro $user receberá alganet e $documentPath receberá um array com o 
  seguinte conteúdo [foo,bar,baz,anything].
 2. Os parâmetros catch-all são definidos por dois astericos: `/**`.
 3. Parametros catch-all devem aparecer apenas no fim do path. Os asteriscos duplos em outras posições serão sempre convertidos para asteriscos simples.
 4. Catch-all parameters will match **after** any other route that matches
    the same pattern.

### Route Matching
[Top][]

As coisas podem se tornar muito complexas rapidamente. Temos rotas simples, rota com parâmetros, parâmetros opcionais e parâmetros catch-all. Uma regra simples que se deve ter em mente é que o Respect/Rest coincide com as rotas a partir do mais especifico ao mais genérico.

  * Rotas com mais `/` são mais específicas e serão correspondidas primeiro.
  * Rotas com parâmetros são menos espcificas que rotas sem parâmetros.
  * Rotas com muitos parâmetros são ainda mais menos especificas do que rotas com menos parâmetros.
  * Rotas com parâmetros catch-all são menos específicas e serão combinadas depois.

Resumindo: A / e o * colocam sua rota no topo da lista de prioridades para combina-los. 

Respect/Rest classifica rotas automáticamente, mas é altamente recomendado organizar rotas do mais específico para o mais genérico, isto visa melhorar o desempenho e manutenção do seu código.

### Matching any HTTP Method
[Top][]

Ás vezes você precisa usar uma rota para proxy request a outro router. Ao usar o método mágico any você poderá passar qualquer requisição HTTP para uma determinada função.
```php
    $r3->any('/users/*', function($userName) {
        /** do anything */
    });
```

 1. Qualquer método HTTP irá corresponder a esta rota.
 2. Você pode descobrir qual é a requisição utilizando o seguinte método padrão do PHP `$_SERVER['REQUEST_METHOD']`

### Class Controllers
[Top][]

O método `any` é extremamente útil para vincular classes para os controladores, uma das caracteristicas mais importantes do Respect/Rest.:
```php
    use Respect\Rest\Routable;

    class MyArticle implements Routable {
        public function get($id) { }
        public function delete($id) { }
        public function put($id) { }
    }

    $r3->any('/article/*', 'MyArticle');
```

  1. Esta rota vai ligar os métodos de classe para os métodos HTTP correspondentes.
  2. Os parâmetros serão enviados para os métodos das classes, tais como os retornos apresentado nos exemplos anteriores.
  3. Os controladores são lazy loaders e persistentes. A classe MyArticle será instanciada somente quando uma rota corresponder a um de seus métodos, e esta instância será reutilizada em callbacks subsequentes(redirecionamentos, etc.)
  4. As classes devem implementar a interface Respect\Rest\Routable por questões de segurança. 
     (Imagine someone mapping HTTP to a PDO class automatically, that wouldn't be right).

Passar argumentos do construtor para a classe também é válido:
```php
    $r3->any('/images/*', 'ImageController', array($myImageHandler, $myDb));
```

  1. O exemplo acima vai passar `$myImageHandler` e `$myDb` para o construtor da classe 
     *ImageController*.

Você também pode instaciar a classe a si mesma, se assim desejar:
```php
    $r3->any('/downloads/*', $myDownloadManager);
```

  1. O exemplo acima irá atribuir  `$myDownloadManager` existente como um controlador.
  2. Esta instância também é reutilizada pelo  Respect\Rest

E você ainda pode utilizar um factory ou DI para construir a classe do controlador:
```php
    $r3->any('/downloads/*', 'MyControllerClass', array('Factory', 'getController'));
```

  1. O exemplo acima irá utilizar a classe  MyController retornado por Factory::getController
  2. Esta instância também é reutilizada pelo Respect\Rest .
  3. O terceiro parâmetro é qualquer variável que pode ser chamada, assim você pode colocar uma closure para construir um exemplo, se assim desejar.

### Routing Streams
[Top][]

Em muitos casos você necessita de rotas para servir usuários de streams. O roteador não necessita primeiro lidar com arquivos grandes ou precisa esperar o stream terminar antes de servi-lo.
```php
    $r3->get('/images/*/hi-res', function($imageName) {
        header('Content-type: image/jpg');
        return fopen("/path/to/hi/images/{$imageName}.jpg", 'r');
    });
```
O exemplo acima irá redirecionar o arquivo diretamente para o navegador sem mantê-lo na memória.

Atenção: Nós criamos uma vunerabilidade de segurança no exemplo: Passando um parâmetro diretamente para um handle fopen. Por favor valide todos os parâmetros de entrada do usuário antes de usá-los. Isto foi somente para uma demonstração!
### Routing Static Values
[Top][]

Sem surpresas aqui. Você pode fazer uma rota retornar uma cadeia de caracteres simples:
```php
    $r3->get('/greetings', 'Hello!');
```

### Forwarding Routes
[Top][]

Respect\Rest possui um mecanismo de encaminhamento interno. Primeiramente é preciso entender que cada declaração de rota retorna uma instância:
```php
    $usersRoute = $r3->any('/users', 'UsersController');
```

Em seguida, você pode usar (`use`) e retornar(`return`) esta rota em outro script:
```php
    $r3->any('/premium', function($user) use ($db, $usersRoute) {
        if (!$db->userPremium($user)) {
          return $usersRoute;
        }
    });
```
O exemplo acima irá redirecionar o usuário para uma outra rota quando o mesmo não tiver permissões suficientes.

### When Routine (if)
[Top][]

Respect\Rest usa uma abordagem diferente para validar os parâmetros de rota:
```php
    $r3->get('/documents/*', function($documentId) {
        /** do something */
    })->when(function($documentId) {
        return is_numeric($documentId) && $documentId > 0;
    });
```

  1. Isso irá corresponder a rota somente se a chamada de retorno for a correspondente.
  2. O parâmetro  `$documentId` deve ter o mesmo nome da ação e da condição.(Mas necessariamente precisar parecer na mesma ordem.)
  3. Você pode especificar mais de um parâmetro de retorno por cada condição.
  4. Você pode especificar mais de um callback: `when($cb1)->when($cb2)->when($etc)`
  5. Condições também irão sincronizar com bind parâmetros de classes e métodos de instância.

Isto traz a possibilidade de cada usuário validar parâmetros usando qualquer rotina personalizada e não apenas os tipos de dados como o `int` ou `string`.

É altamente recomendado utilizar uma biblioteca poderosa de validação neste momento. Veja o
[Respect\Validation](http://github.com/Respect/Validation).
```php
    $r3->get('/images/*/hi-res', function($imageName) {
        header('Content-type: image/jpg');
        return fopen("/path/to/hi/images/{$imageName}.jpg", 'r');
    })->when(function($imageName) {
        /** Using Respect Validation alias to `V` */
        return V::alphanum(".")->length(5,155)
                ->noWhitespace()->validate($imageName);
    });
```

### By Routine (before)
[Top][]

Às vezes você precisa executar algo antes que uma rota execute seu trabalho. Isso é útil para registros, autenticações e atividades similares.
```php
    $r3->get('/artists/*/albums/*', function($artistName, $albumName) {
        /** do something */
    })->by(function($albumName) use ($myLogger) {
        $myLogger->logAlbumVisit($albumName);
    });
```

  1. Isto irá executar o callback definido antes da ação da rota que precisa corresponder a uma rota.
  2. Os parâmetros são sincronizado pelo nome e não pela ordem, como com `when`.
  3. Você pode especificar mais de um parâmetro de retorno por chamada de proxy.
  4. Você pode especificar mais de um proxy: `by($cb1)->by($cb2)->by($etc)`
  5. Um `return false` irá parar a execução de qualquer proxy seguinte, bem como as ações da rota.
  6. Proxies também irão sincronizar com parâmetros de bind class e métodos de instância.

Se sua rotina retorna false, então o método/função de rota não será processado. Se você retornar uma instância de outra rota, então o próximo método interno será executado.

### Through Routine (after)
[Top][]

Similar ao `->by`, porém é executado após a rota fazer seu trabalho. No exemplo a seguir, nós estamos mostrando algo semelhante para invalidar um cache depois de enviar e salvar algumas informações novas.
```php
    $r3->post('/artists/*/albums/*', function($artistName, $albumName) {
        /** save some artist info */
    })->through(function() use($myCache) {
        $myCache->clear($artistName, $albumName);
    });
```

  1. `by` proxies serão executadas antes das ações da rota, through proxies serão executadas depois.
  2. Você é livre para usá-los em conjunto ou separados.
  3. `through` também pode receber parâmetros por nome.

O exemplo acima permite que você faça alguma coisa com base nos parâmetros da rota, mas quando processado após a execução da rota, é desejavel para processar sua saida também. Isto pode ser conseguido com nested closure.
```php
    $r3->any('/settings', 'SetingsController')->through(function(){
        return function($data) {
            if (isset($settings['admin_user'])) {
                unset($settings['admin_user']);
            }
            return $data;
        };
    });
```
Ao usar as rotinas você é encorajado a separar a lógica do controlador dos componentes. Você pode reutizá-los

### Controller Splitting
[Top][]

Ao usar as rotinas você é encorajado a separar a lógica do controlador dos componentes. Você pode reutizá-los
```php
    $logRoutine = function() use ($myLogger, $r3) {
        $myLogger->logVisit($r3->request->path);
    };

    $r3->any('/users', 'UsersController')->by($logRoutine);
    $r3->any('/products', 'ProductsController')->by($logRoutine);
```

É uma maneira simples de rotinas para cada rota no router.
```php
    $r3->always('By', $logRoutine);
```

Você pode usar o sync param para obter vantagem disto:
```php
    $r3->always('When', function($user=null) {
        if ($user) {
          return strlen($user) > 3;
        }
    });
    
    $r3->any('/products', function () { /***/ });
    $r3->any('/users/*', function ($user) { /***/ });
    $r3->any('/users/*/products', function ($user) { /***/ });
    $r3->any('/listeners/*', function ($user) { /***/ });
```

Uma vez que existe três rotas com parâmetros $user, when irá verificar automaticamente pelo seu nome.

### Content Negotiation
[Top][]

Respect\Rest atualmente oferece 4 tipos distintos de Accept Header: Mimetype, Encoding, Language and Charset. Caso de uso:
```php
    $r3->get('/about', function() {
        return array('v' => 2.0);
    })->acceptLanguage(array(
        'en' => function($data) { return array("Version" => $data['v']); },
        'pt' => function($data) { return array("Versão"  => $data['v']); }
    ))->accept(array(
        'text/html' => function($data) {
            list($k,$v)=each($data);
            return "<strong>$k</strong>: $v";
        },
        'application/json' => 'json_encode'
    ));
```

Como em cada rotina, rotinas conneg(Content Negotiation) são executadas na mesma ordem que você anexar nas rotas. Você pode usar `->always` para aplicar esta rotina para cada rota no Router.

Por favor, note que ao retornar streams, também são chamados de conneg routines. Você pode aproveitar isso ao processar streams. O exemplo hardcore a seguir, usa o deflate encoding, diretamente no browser:
```php
    $r3->get('/text/*', function($filename) {
      return fopen('data/'.$filename, 'r+');
    })->acceptEncoding(array(
        'deflate' => function($stream) {
            stream_filter_append($stream, 'zlib.deflate', STREAM_FILTER_READ);
            return $stream; /** now deflated on demand */
        }
    ));
```

Ao aplicarmos rotinas conneg em várias rotas que podem retornar streams você(realmente) deve verificar se há `is_resource()` antes de fazer qualquer coisa.

### Basic HTTP Auth
[Top][]

Suporte básico para autenticação HTTP já é implementado como rotina:
```php
    $r3->get('/home', 'HomeController')->authBasic('My Realm', function($user, $pass) {
        return $user === 'admin' && $pass === 'p4ss';
    }); 
```

Você receberá um usuário e senha submetidos pelo usuário e você só precisar retornar true ou false. TRUE indica que o usuário pode ser autenticado.

Respect\Rest irá lidar com o fluxo de autenticação, enviando os cabeçalhos apropriados, quando não autenticado. Você também pode retornar outras rotas, que irá atuar como uma frente interna(consulte a seção foward above acima).

### Filtering Browsers
[Top][]

Abaixo está um exemplo desmonstrativo de como bloquear acesso de mobile devices:
```php
    $r3->get('/videos/*', 'VideosController')->userAgent(array(
        'iphone|android' => function(){
            header('HTTP/1.1 403 Forbidden');
            return false; /** do not process the route. */
        }
    ));
```

Você pode passar vários itens no array como nas rotinas conneg. As chaves do array é uma correspondência de expressão regular sem delimitadores.

### Input Content-Type (input data)
[Top][]

Observe que isso atualmente não é implementado.

Por padrão os formulários HTML enviam dados via POST como `multipart/form-data`, mas clientes API podem enviar em outros formatos.Requisições PUT muitas vezes enviam outros tipos de MIME. Você pode pre-processar esses dados antes de fazer qualquer coisa:
```php
    $r3->post('/timeline', function() {
        return file_get_contents('php://input');
    })->contentType(array(
        'multipart/form-data' => function($input) {
            parse_str($input, $output);
            return $output;
        },
        'application/json' => function($input) {
            return my_json_converter($input);
        },
        'text/xml' => function($input) {
            return my_xml_converter($input);
        },
    ));
```

### HTTP Errors
[Top][]

Respect\Rest atualmente lida com os seguinte tipos de erros por padrão:

  * 404, quando não existe nenhuma correspodência nos paths da rota.
  * 401, quando o cliente envia uma solicitação de não autenticado para uma rota usando a rotinaauthBasic
  * 405, quando um caminho correpondente for encontrado, mas o método não for especificado.
  * 400, quando a validação when falhar.
  * 406, quando o route path e o método batem mas o content-negotiation não.

### RESTful Extras
[Top][]

  * Uma requisição HEAD trabalha automáticamente enviando os cabeçalhos de requisições GET sem corpo. Você pode substituir esse comportamento declarando rotas personalizadas  `head`.
  * Uma requisição OPTIONS para `*` ou qualquer caminho de rota retorna o cabeçalho `Allow` correto.
  * Quando retorna 405, os cabeçalho Allow também estão definidos corretamente.

### Anti-Patterns
[Top][]

  * Você pode definir  `$r3->methodOverriding = true` para permitir que `?_method=ANYMETHOD` como o URI para substituir métodos HTTP padrões. Isto é `false` por default

### Your Own Routines
[Top][]

Rotinas são classes no namespace Respect\Rest\Routines. Mas você pode usar suas próprias rotinas usando a instância:

```php
    $r3->get('/greetings', 'Hello World')->appendRoutine(new MyRoutine);
```

No exemplo acima, `MyRoutine` é uma rotina de usuário fornecida e declarada em uma router. Rotinas personalizadas têm a opção de várias interface diferentes, que podem ser implementadas:

  * IgnorableFileExtension - Instrui o roteador a ignorar a extensão de arquivo em pedido.
  * ParamSynced - Sincroniza parâmetros com a rota function/method.
  * ProxyableBy - Instrui o router a executar o método `by()` antes da rota.
  * ProxyableThrough - Instrui o router a executar o método through() depois da rota.
  * ProxyableWhen - Instrui o router a executar o método when para validar a partida da rota.
  * Unique - Faz a rotina ser substituida, não anexada, se mais de um é declarado para o mesmo tipo.
  
Você pode usar qualquer combinação acima mas terá de implementar a interface Routinable.

### Error Handling
[Top][]

O Respect\Rest oferece duas maneiras de se tratar erros. A primeira é usando Exception Routes:

```php
    $r3->exceptionRoute('InvalidArgumentException', function (InvalidArgumentException $e) {
        return 'Sorry, this error happened: '.$e->getMessage();
    });
```

Sempre que uma exceção não capturada aparece em qualquer rota, será presa e encaminhada para outra rota ao lado. Da mesma forma, existe uma rota de erros PHP:


```php
    $r3->errorRoute(function (array $err) {
        return 'Sorry, this errors happened: '.var_dump($err);
    });
```

[Top]: #navigation
[Configuration]: #configuration
[Dispatching]: #dispatching
[Simple Routing]: #simple-routing
[Parameters]: #using-parameters
[Catch-all]: #catch-all-parameters
[Matching]: #route-matching
[Methods]: #matching-any-http-method
[Controllers]: #class-controllers
[Streams]: #routing-streams
[Static]: #routing-static-values
[Forwarding]: #forwarding-routes
[When]: #when-routine-if
[By]: #by-routine-before
[Through]: #through-routine-after
[Controller Splitting]: #controller-splitting
[Conneg]: #content-negotiation
[Basic Auth]: #basic-http-auth
[User-Agent]: #filtering-browsers
[Content-Type]: #input-content-type-input-data
[HTTP Errors]: #http-errors
[RESTful Extras]: #restful-extras
[Anti-Patterns]: #anti-patterns
[Own Routines]: #your-own-routines
[Error Handling]: #error-handling
