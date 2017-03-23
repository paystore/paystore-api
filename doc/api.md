# Integração com Aplicação de Pagamentos via API

Uma das formas de se integrar com a aplicação de pagamentos é via [IPC] (https://developer.android.com/guide/components/aidl.html). Para isto, é fornecida uma biblioteca, a `payment-api.aar`, contendo todo o código necessário a ser usada para tais chamadas.

Usando esta API, é possível realizar pagamentos, confirmá-los ou cancelá-los (desfazer), e estorná-los. Estes pagamentos podem ser solicitados com valores pré-definidos ou valores em aberto a serem entrados pelo operador do terminal, uma lista de tipos de pagamentos (débito, crédito à vista, crédito parcelado, etc.) permitidos ou sem tal restrição, confiorme especificação a seguir.

Ainda que esta integração se dê através de uma API, a aplicação de pagamentos pode exibir informações na interface do terminal, tais como mensagens (e.g., "Insira ou Passe o Cartão"), ou mesmo solicitar informações do operador (e.g., CVV). Assim sendo, durante a realização de qualquer operação, a aplicação que solicitou a operação não deve interagir com a interface do dispositivo até que a operação seja concluída.

A seguir, temos a especificação detalhada das operações disponíveis.

Para integração com a API de pagamentos, é fornecida a interface `PaymentClient`.
  
## Métodos
  
| Assinatura | Descrição |
| -------- | -------- |
| [`void startPayment(PaymentRequestDTO paymentRequest, PaymentCallback paymentCallback)`](#startpayment)| Realiza o processo de autorização de pagamento. |
| [`void confirmPayment(Long paymentId, PaymentCallback paymentCallback)`](#confirmpayment) | Confirma uma autorização de pagamento realizada anteriormente.   |
| [`void cancelPayment(Long paymentId, PaymentCallback paymentCallback)`](#cancelpayment) | Desfaz uma autorização de pagamento realizada anteriormente. |
| [`void reversePayment(ReversePaymentRequestDTO paymentRequest, PaymentCallback paymentCallback)`](#reversepayment) | Realiza o processo de estorno de pagamento.  |
| [`void cancelReversePayment(Long paymentId, PaymentCallback paymentCallback)`](#cancelReversepayment) | Desfaz uma solicitação de estorno de pagamento.  |

### `startPayment()`

Este método deve ser chamado quando se deseja fazer uma solicitação de autorização de pagamento. Durante sua execução, os dados do pagamento serão validados, informações adicionais serão solicitadas ao operador (e.g. senha e CVV), e a autorização junto à adquirente será feita.

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `PaymentRequestDTO` | Sim | Objeto de transferência de dados que conterá as informações da requisição do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de pagamento.   |
    
**Detalhe dos Parâmetros**  
  
_request (PaymentRequestDTO)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor do pagamento solicitado. Caso não seja preenchido (null), a interface solicitará o valor do operador. |
| `paymentTypes` | `List<PaymentType>` | Não | Tipos de pagamentos (Débito, Crédito, Voucher, etc.) permitidos para este pagamento. Caso seja vazio ou seja null, significa que todos os tipos são permitidos. Caso contenha apenas um, este tipo será o utilizado (se possível) e não será perguntado nada ao operador. |
| `installments` | `Integer` | Não | Quantidade de parcelas. Usado apenas para tipos de pagamentos que suportem parcelamento e neste caso é obrigatório. Valor deve ser entre 2 e 99. | 
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de pagamento. Não deve se repetir. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 

_callback (PaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `PaymentDTO.value` | `BigDecimal` | Sim | Valor do pagamento. Este é o valor que foi aprovado pela adquirente. Deve ser validado sempre na resposta, ainda que tenha sido passado como parâmetro, pois há adquirentes que, para algumas situações, aprovam valores diferentes dos solicitados. |
| `PaymentDTO.paymentType` | `PaymentType` | Sim | Tipo de pagamento (Débito, Crédito, Voucher, etc.) usado no pagamento. |
| `PaymentDTO.installments` | `Integer` | Não | Quantidade de parcelas do pagamento. |
| `PaymentDTO.acquirer` | `String` | Sim | Adquirente que autorizou o pagamento. |
| `PaymentDTO.paymentId` | `String` | Sim | Identificador da transação para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `PaymentDTO.brand` | `String` | Sim | Bandeira do cartão usado no pagamento. |
| `PaymentDTO.bin` | `String` | Sim | Bin do cartão usado no pagamento. |
| `PaymentDTO.panLast4Digits` | `String` | Sim | Últimos 4 dígitos do PAN do cartão usado na transação. |
| `PaymentDTO.captureType` | `CaptureTypeDTO` | Sim | Forma de captura do cartão usado na transação. |
| `PaymentDTO.paymentDate` | `Date` | Sim | Data/hora do pagamento para a aplicação de pagamentos. |
| `PaymentDTO.acquirerId` | `String` | Sim | Identificador da transação para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do pagamento com a transação integrada. |
| `PaymentDTO.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `PaymentDTO.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `PaymentDTO.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `PaymentDTO.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `PaymentDTO.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorDTO.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorDTO.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorDTO.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

##### Exemplo

```java
public class MyActivity extends Activity implements PaymentClient.PaymentCallback {

    private PaymentClient paymentClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_payment);
    
        paymentClient = new PaymentClientImpl();
    }

    @Override
    protected void onResume() {
        super.onResume();
        paymentClient.bind(this);
    }

    @Override
    protected void onPause() {
         try {
            paymentClient.unbind(this);
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        super.onPause();
    }

    public void doExecute(){
        PaymentRequestDTO request = new PaymentRequestDTO();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.startPayment(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(PaymentDTO payment) {
        Log.i(TAG, payment.toString());
    }
}
```

### `confirmPayment()`

Este método deve ser chamado para confirmar uma transação anteriormente autorizada. Esta transação deve não ter sido confirmada ou desfeita ainda e deve ter sido autorizada (não negada) previamente. Após a execução desta confirmação, a transação só poderá ser cancelada através de uma operação de estorno. 

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `paymentId` | `Long` | Sim | Identificador da transação que será confirmada para a aplicação de pagamentos. |
| `credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorDTO.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorDTO.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorDTO.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

##### Exemplo

```java
public class MyActivity extends Activity implements PaymentClient.PaymentCallback {

    private PaymentClient paymentClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_payment);
    
        paymentClient = new PaymentClientImpl();
    }

    @Override
    protected void onResume() {
        super.onResume();
        paymentClient.bind(this);
    }

    @Override
    protected void onPause() {
        try {
            paymentClient.unbind(this);
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        super.onPause();
    }

    public void doExecute(){
        PaymentRequestDTO request = new PaymentRequestDTO();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.startPayment(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(PaymentDTO payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.confirmPayment(payment.getPaymentId(),
                                         new Credentials("demo-app", "TOKEN-KEY-DEMO")
                                         new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(PaymentDTO payment) {
                    Log.i(TAG, payment.toString());
                }
            
            
            });
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }
}
```

### `cancelPayment()`

Este método deve ser chamado para desfazer uma transação anteriormente autorizada. Esta transação deve não ter sido confirmada ou desfeita ainda e deve ter sido autorizada (não negada) previamente. Após a execução deste desfazimento, a transação não mais poderá ser confirmada. 

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `paymentId` | `Long` | Sim | Identificador da transação que será desfeita para a aplicação de pagamentos. |
| `credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorDTO.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorDTO.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorDTO.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

##### Exemplo

```java
public class MyActivity extends Activity implements PaymentClient.PaymentCallback {

    private PaymentClient paymentClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_payment);
    
        paymentClient = new PaymentClientImpl();
    }

    @Override
    protected void onResume() {
        super.onResume();
        paymentClient.bind(this);
    }

    @Override
    protected void onPause() {
        try {
            paymentClient.unbind(this);
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        super.onPause();
    }

    public void doExecute(){
        PaymentRequestDTO request = new PaymentRequestDTO();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.startPayment(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(PaymentDTO payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.cancelPayment(payment.getPaymentId(), 
                                        new Credentials("demo-app", "TOKEN-KEY-DEMO"), 
                                        new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(PaymentDTO payment) {
                    Log.i(TAG, payment.toString());
                }
            
            
            });
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }
}
```

### `reversePayment()`

Este método deve ser chamado quando se deseja fazer uma solicitação de estorno de pagamento. Durante sua execução, os dados do estorno serão validados, informações adicionais serão solicitadas ao operador (e.g. cartão) e a autorização junto à adquirente será feita.

Note que a transação de estorno não possui confirmação, mas apenas desfazimento. Assim, a confirmação ocorrerá naturalmente com o não envio do desfazimento, a depender do comportamento de cada adquirente.

Também a depender do comportamento de cada adquirente, é possível que não haja desfazimento para a transação de estorno para uma determinada adquirente. Neste caso, estornos aprovados retornarão o valor _false_ no campo "ReversePaymentDTO.cancelable". Além disto, caso seja chamado o método `cancelReversePayment()`, um erro específico será retornado informando que não é possível executar tal operação (vide [Códigos de Resposta](#codigos-de-resposta)).

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `ReversePaymentRequestDTO` | Sim | Objeto de transferência de dados que conterá as informações da requisição do estorno do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `ReversePaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de estorno.   |
    
**Detalhe dos parâmetros**  
  
_request (ReversePaymentRequestDTO)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor da transação a ser estornada. Caso não seja preenchido (null), a interface solicitará o valor do operador. Esta informação é utilizada para validar a integridade da transação que está sendo estornada. |
| `paymentId` | `Long` | Sim | Identificador da transação que será estornada para a aplicação de pagamentos. |
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de estorno. Não deve se repetir. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 

_callback (ReversePaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `ReversePaymentDTO.paymentId` | `String` | Sim | Identificador da transação de estorno para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `ReversePaymentDTO.acquirerId` | `String` | Sim | Identificador da transação de estorno para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do estorno com a transação integrada. |
| `ReversePaymentDTO.cancelable` | `Boolean` | Sim | _True_, caso esta transação possa ser desfeita. _False_ caso contrário. |
| `ReversePaymentDTO.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `ReversePaymentDTO.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `ReversePaymentDTO.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `ReversePaymentDTO.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `ReversePaymentDTO.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorDTO.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorDTO.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorDTO.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

##### Exemplo

```java
public class MyActivity extends Activity implements PaymentClient.PaymentCallback {

    private PaymentClient paymentClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_payment);
    
        paymentClient = new PaymentClientImpl();
    }

    @Override
    protected void onResume() {
        super.onResume();
        paymentClient.bind(this);
    }

    @Override
    protected void onPause() {
        try {
            paymentClient.unbind(this);
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        super.onPause();
    }

    public void doExecute(){
        ReversePaymentRequestDTO request = new ReversePaymentRequestDTO();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        request.setPaymentId("999999");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.reversePayment(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(ReversePaymentDTO payment) {
        Log.i(TAG, payment.toString());
    }
}
```
### `cancelReversePayment()`

Este método deve ser chamado para desfazer uma transação de estorno anteriormente autorizada. Esta transação deve não ter sido desfeita ainda e deve ter sido autorizada (não negada) previamente. 

Como dito na descrição de [`reversePayment()`](#reversepayment), é possível que não haja desfazimento para a transação de estorno para uma determinada adquirente. Assim, caso seja chamado o método `cancelReversePayment()`, um erro específico será retornado informando que não é possível executar tal operação (vide [Códigos de Resposta](#codigos-de-resposta)).


**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `paymentId` | `Long` | Sim | Identificador da transação que será desfeita para a aplicação de pagamentos. |
| `credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorDTO.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorDTO.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorDTO.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

##### Exemplo

```java
public class MyActivity extends Activity implements PaymentClient.PaymentCallback {

    private PaymentClient paymentClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_payment);
    
        paymentClient = new PaymentClientImpl();
    }

    @Override
    protected void onResume() {
        super.onResume();
        paymentClient.bind(this);
    }

    @Override
    protected void onPause() {
        try {
            paymentClient.unbind(this);
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        super.onPause();
    }

    public void doExecute(){
        ReversePaymentRequestDTO request = new ReversePaymentRequestDTO();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        request.setPaymentId("999999");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.reversePayment(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(ReversePaymentDTO payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.cancelReversePayment(payment.getPaymentId(), 
                                               new Credentials("demo-app", "TOKEN-KEY-DEMO"),
                                               new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(PaymentDTO payment) {
                    Log.i(TAG, payment.toString());
                }
            
            
            });
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }
}
```

# Integração com Aplicação de Pagamentos via Intent

As _Intents_ permitem iniciar uma atividade em outro aplicativo descrevendo uma ação simples que você quer executar em um objeto [Intent](https://developer.android.com/reference/android/content/Intent.html). O aplicativo de pagamentos expôe  _Intents_ para a realização de pagamento e estorno. No caso de pagamento, a _Intent_ possibilita a exibição da tela de digitação do valor e uso das suas diversas funcionalidades, tal como realização de vários pagamentos (_split_). Para estorno, a _Intent_ exibe tela para a localização da transação a ser estornada além das telas que compões a realização do estorno em si.

## Solicitações de _Intent_

Para iniciar o aplicativo de pagamentos com uma _Intent_, é preciso antes criar um objeto _Intent_ especificando sua ação e o pacote.

- **Action**: Todas as _Intents_ são chamadas com uma _action_ específica para cada fim, conforme descrição a seguir.
- **Package**: Chame setPackage("br.com.phoebus.android.payments") para garantir que o aplicativo de pagamentos processe as intenções. Se o pacote não está definido, o sistema determina quais aplicativos podem processar a _Intent_.

Após criar a _Intent_, você pode solicitar que o sistema inicie o aplicativo relacionado de diversas formas. Um método comum é passar a _Intent_ ao método `startActivity()`. O sistema lança o aplicativo necessário — neste caso, o de pagamentos — e inicia a _Activity_ correspondente.

```java
// Create an Intent with the action to ACTION_START_PAYMENT
Intent paymentIntent = new Intent(Intents.action.ACTION_START_PAYMENT);

// Make the Intent explicit by setting the Payment App package
paymentIntent.setPackage("br.com.phoebus.android.payments");

// Attempt to start an activity that can handle the Intent
startActivity(paymentIntent);
```

**Atenção**: É muito importante atentar-se que pagamentos e estornos feitos via _Intents_ são automaticamente confirmados. Pagamentos só podem ser desfeitos via estorno ([API](#reversepayment) ou [Intent](#action-action_reverse_payment)).

Constantes para os _actions_ e _extras_ podem ser encontrados em:

- `br.com.phoebus.android.payments.api.utils.Intents.action`
- `br.com.phoebus.android.payments.api.utils.Intents.extra`

## Action `ACTION_START_PAYMENT`

### Parâmetros de Entrada

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `EXTRA_PAYMENT_VALUE` | `BigDecimal` | Não | Valor do pagamento solicitado. Caso não seja preenchido (null), a interface solicitará o valor do operador. |
| `EXTRA_PAYMENT_TYPES` | `List<PaymentType>` | Não | Tipos de pagamentos (Débito, Crédito, Voucher, etc.) permitidos para este pagamento. Caso seja vazio ou seja null, significa que todos os tipos são permitidos. Caso contenha apenas um, este tipo será o utilizado (se possível) e não será perguntado nada ao operador. |
| `EXTRA_PAYMENT_ALLOWS_MULTIPLE_PAYMENTS` | `Boolean` | Não | _True_ (_default_) caso o operador possa fazer _split_ do pagamento, _false_ caso contrário. | 
| `EXTRA_PAYMENT_ALLOWS_CASH` | `Boolean` | Não | _True_ caso seja permitido registrar pagamentos em dinheiro, _false_ (_default_) caso contrário. | 
| `EXTRA_PAYMENT_SHOW_RECEIPTS` | `Boolean` | Não | _True_ (_default_) caso seja para a aplicação de pagamentos exibir os comprovantes, _false_ caso contrário. |
| `EXTRA_PAYMENT_APP_TRANSACTION_ID` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de pagamento. Não deve se repetir. |
| `EXTRA_PAYMENT_APPLICATION_INFO` | `ApplicationInfo` | Sim | Vide parâmetros de [startPayment()](#startpayment) para mais informações. |

### Parâmetros de Saída

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `EXTRA_PAYMENT_RESPONSES` | `List<PaymentDTO>` | Sim | Lista contendo os pagamentos realizados. Para mais detalhes sobre `PaymentDTO`, vide [startPayment()](#startpayment) ou documentação da classe. |

## Action `ACTION_REVERSE_PAYMENT`

### Parâmetros de Entrada

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `EXTRA_PAYMENT_PAYMENT_ID` | `String` | Não | Identificador da transação que está sendo estornada para a aplicação de pagamento. Caso não seja passado, uma interface de busca padrão da aplicação de pagamentos será exibida. |
| `EXTRA_PAYMENT_APP_TRANSACTION_ID` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de estorno. Não deve se repetir. |
| `EXTRA_PAYMENT_APPLICATION_INFO` | `ApplicationInfo` | Sim | Vide parâmetros de [reversePayment()](#reversepayment) para mais informações. |

### Parâmetros de Saída

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `EXTRA_PAYMENT_REVERSE_RESPONSE` | `ReversePaymentDTO` | Sim | Informações sobre o estorno realizado. Para mais detalhes sobre `ReversePaymentDTO`, vide [reversePayment()](#reversepayment) ou documentação da classe. |


# Integração com Aplicação de Pagamentos via Content Povider

_[Desrever aqui!]_

# Códigos de Resposta

| Código | Descrição | Operações |
| -------| --------- | --------- |
| 01     | Transação negada pela adquirente. | `startPayment` |
| 02     | Transação negada pelo cartão. | `startPayment` |
| 03     | Operação cancelada pelo operador. | `startPayment` e `reversePayment` |
| 04     | Pagamento não encontrado. | `confirmPayment`, `cancelPayment`, `reversePayment` e `cancelReversePayment` |
| 05     | Problerma na comunicação com o aplicativo de pagamento. | Todas |
| 06     | Operação não disponível na adquirente. | `cancelReversePayment` |
| 07     | Problema de comunicação com a adquirente. | Todas |
| 99     | Problema Interno. | Todas |

