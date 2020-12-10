# Integração com Aplicação de Pagamentos via API

Uma das formas de se integrar com a aplicação de pagamentos é via [IPC] (https://developer.android.com/guide/components/aidl.html). Para isto, é fornecida uma biblioteca, a `payments-api-1.2.6.0.aar`, contendo todo o código necessário a ser usada para tais chamadas.

Usando esta API, é possível realizar pagamentos, confirmá-los ou cancelá-los (desfazer), e estorná-los. Estes pagamentos podem ser solicitados com valores pré-definidos ou valores em aberto a serem entrados pelo operador do terminal, uma lista de tipos de pagamentos (débito, crédito à vista, crédito parcelado, etc.) permitidos ou sem tal restrição, confiorme especificação a seguir.

Ainda que esta integração se dê através de uma API, a aplicação de pagamentos pode exibir informações na interface do terminal, tais como mensagens (e.g., "Insira ou Passe o Cartão"), ou mesmo solicitar informações do operador (e.g., CVV). Assim sendo, durante a realização de qualquer operação, a aplicação que solicitou a operação não deve interagir com a interface do dispositivo até que a operação seja concluída.

A seguir, temos a especificação detalhada das operações disponíveis.

Para integração com a API de pagamentos, é fornecida a interface `PaymentClient`.
  
## Métodos
  
| Assinatura | Descrição |
| -------- | -------- |
| [`void startPayment(PaymentRequest paymentRequest, PaymentCallback paymentCallback)`](#startpayment)| Realiza o processo de autorização de pagamento. ( DEPRECATED : Utilizar startPaymentV2 ) |
| [`void startPaymentV2(PaymentRequestV2 paymentRequest, PaymentCallback paymentCallback)`](#startpaymentV2)| Realiza o processo de autorização de pagamento. |
| [`void confirmPayment(String paymentId, PaymentCallback paymentCallback)`](#confirmpayment) | Confirma uma autorização de pagamento realizada anteriormente.   |
| [`void cancelPayment(String paymentId, PaymentCallback paymentCallback)`](#cancelpayment) | Desfaz uma autorização de pagamento realizada anteriormente. |
| [`void reversePayment(ReversePaymentRequest paymentRequest, PaymentCallback paymentCallback)`](#reversepayment) | Realiza o processo de estorno de pagamento.  ( DEPRECATED : Utilizar reversePaymentV2 ) |
| [`void reversePaymentV2(ReversePaymentRequestV2 paymentRequest, PaymentCallback paymentCallback)`](#reversepaymentV2) | Realiza o processo de estorno de pagamento.  |
| [`void cancelReversePayment(String paymentId, PaymentCallback paymentCallback)`](#cancelReversepayment) | Desfaz uma solicitação de estorno de pagamento.  |
| [`void setTheme(String theme, PaymentCallback paymentCallback)`](#setTheme) | Define um tema para a aplicação de Pagamentos.  |
| [`void startExtraction(PaymentCallback paymentCallback)`](#startExtraction) | Faz o upload dos dados do Payments e das adquirentes para a AWS. |

### `startPayment()`  **( DEPRECATED : Utilizar startPaymentV2 )**

Este método deve ser chamado quando se deseja fazer uma solicitação de autorização de pagamento. Durante sua execução, os dados do pagamento serão validados, informações adicionais serão solicitadas ao operador (e.g. senha e CVV), e a autorização junto à adquirente será feita.

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `PaymentRequest` | Sim | Objeto de transferência de dados que conterá as informações da requisição do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de pagamento.   |
    
**Detalhe dos Parâmetros**  
  
_request (PaymentRequest)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor do pagamento solicitado. Caso não seja preenchido (null), a interface solicitará o valor do operador. |
| `paymentTypes` | `List<PaymentType>` | Não | Tipos de pagamentos (Débito, Crédito, Voucher, etc.) permitidos para este pagamento. Caso seja vazio ou seja null, significa que todos os tipos são permitidos. Caso contenha apenas um, este tipo será o utilizado (se possível) e não será perguntado nada ao operador. |
| `installments` | `Integer` | Não | Quantidade de parcelas. Usado apenas para tipos de pagamentos que suportem parcelamento e neste caso é obrigatório. Valor deve ser entre 2 e 99. | 
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de pagamento. Não deve se repetir. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 
| `showReceiptView` | `IGNORADO` | Não | A Solução sempre irá imprimir o comprovante depois que a [`confirmação`](#confirmpayment) for executada.  | 

_callback (PaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `Payment.value` | `BigDecimal` | Sim | Valor do pagamento. Este é o valor que foi aprovado pela adquirente. Deve ser validado sempre na resposta, ainda que tenha sido passado como parâmetro, pois há adquirentes que, para algumas situações, aprovam valores diferentes dos solicitados. |
| `Payment.paymentType` | `PaymentType` | Sim | Tipo de pagamento (Débito, Crédito, Voucher, etc.) usado no pagamento. |
| `Payment.installments` | `Integer` | Não | Quantidade de parcelas do pagamento. |
| `Payment.acquirer` | `String` | Sim | Adquirente que autorizou o pagamento. |
| `Payment.paymentId` | `String` | Sim | Identificador da transação para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `Payment.brand` | `String` | Sim | Bandeira do cartão usado no pagamento. |
| `Payment.bin` | `String` | Sim | Bin do cartão usado no pagamento. |
| `Payment.panLast4Digits` | `String` | Sim | Últimos 4 dígitos do PAN do cartão usado na transação. |
| `Payment.captureType` | `CaptureType` | Sim | Forma de captura do cartão usado na transação. |
| `Payment.paymentStatus` | `PaymentStatus` | Sim | Situação do pagamento. No caso de solitações retornadas com sucesso, esta informação sempre será _PENDING_, requerendo uma confirmação ou desfazimento para a sua conclusão definitiva. |
| `Payment.paymentDate` | `Date` | Sim | Data/hora do pagamento para a aplicação de pagamentos. |
| `Payment.acquirerId` | `String` | Sim | Identificador da transação para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do pagamento com a transação integrada. |
| `Payment.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `Payment.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `Payment.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `Payment.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `Payment.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        PaymentRequest request = new PaymentRequest();
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
    public void onSuccess(Payment payment) {
        Log.i(TAG, payment.toString());
    }
}
```

### `startPaymentV2()`

Este método deve ser chamado quando se deseja fazer uma solicitação de autorização de pagamento. Durante sua execução, os dados do pagamento serão validados, informações adicionais serão solicitadas ao operador (e.g. senha e CVV), e a autorização junto à adquirente será feita.

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `PaymentRequestV2` | Sim | Objeto de transferência de dados que conterá as informações da requisição do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de pagamento.   |
    
**Detalhe dos Parâmetros**  
  
_request (PaymentRequestV2)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor do pagamento solicitado. Caso não seja preenchido (null), a interface solicitará o valor do operador. |
| `paymentTypes` | `List<PaymentType>` | Não | Tipos de pagamentos (Débito, Crédito, Voucher, etc.) permitidos para este pagamento. Caso seja vazio ou seja null, significa que todos os tipos são permitidos. Caso contenha apenas um, este tipo será o utilizado (se possível) e não será perguntado nada ao operador. |
| `installments` | `Integer` | Não | Quantidade de parcelas. Usado apenas para tipos de pagamentos que suportem parcelamento e neste caso é obrigatório. Valor deve ser entre 2 e 99. | 
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de pagamento. Não deve se repetir. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 
| `showReceiptView` | `IGNORADO` | Não | A Solução irá utilizar o valor dos parâmetros `printMerchantReceipt` e `printCustomerReceipt` para executar a impressão depois que a [`confirmação`](#confirmpayment) for executada. | 
| `printMerchantReceipt` | `Boolean` | Não | Indica se o comprovante do estabelecimento deve ser impresso depois da [`confirmação`](#confirmpayment) da transação. O valor padrão é _true_. | 
| `printCustomerReceipt` | `Boolean` | Não | Indica se o comprovante do cliente deve ser impresso depois da [`confirmação`](#confirmpayment) da transação. O valor padrão é _true_. | 
| `tokenizeCard` | `Boolean` | Não | Indica se deve ser feita a tokenização do cartão após a aprovação do pagamento (true) ou não (false). O valor padrão é false.  | 
| `tokenizeEmail` | `String` | Se tokenizeCard for true, sim, caso contrário, não. | E-mail do portador do cartão. Se “tokenizeCard” for false, este parâmetro é ignorado. | 
| `tokenizeNationalDocument` | `String` | Não | CPF ou CNPJ do portador do cartão. Se “tokenizeCard” for false, este parâmetro é ignorado. Se for true e não for informado esse parâmetro, então a chamada à API de criação de token no e-commerce também não o utilizará. | 
  

_callback (PaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `Payment.value` | `BigDecimal` | Sim | Valor do pagamento. Este é o valor que foi aprovado pela adquirente. Deve ser validado sempre na resposta, ainda que tenha sido passado como parâmetro, pois há adquirentes que, para algumas situações, aprovam valores diferentes dos solicitados. |
| `Payment.paymentType` | `PaymentType` | Sim | Tipo de pagamento (Débito, Crédito, Voucher, etc.) usado no pagamento. |
| `Payment.installments` | `Integer` | Não | Quantidade de parcelas do pagamento. |
| `Payment.acquirer` | `String` | Sim | Adquirente que autorizou o pagamento. |
| `Payment.paymentId` | `String` | Sim | Identificador da transação para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `Payment.brand` | `String` | Sim | Bandeira do cartão usado no pagamento. |
| `Payment.bin` | `String` | Sim | Bin do cartão usado no pagamento. |
| `Payment.panLast4Digits` | `String` | Sim | Últimos 4 dígitos do PAN do cartão usado na transação. |
| `Payment.captureType` | `CaptureType` | Sim | Forma de captura do cartão usado na transação. |
| `Payment.paymentStatus` | `PaymentStatus` | Sim | Situação do pagamento. No caso de solitações retornadas com sucesso, esta informação sempre será _PENDING_, requerendo uma confirmação ou desfazimento para a sua conclusão definitiva. |
| `Payment.paymentDate` | `Date` | Sim | Data/hora do pagamento para a aplicação de pagamentos. |
| `Payment.acquirerId` | `String` | Sim | Identificador da transação para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do pagamento com a transação integrada. |
| `Payment.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `Payment.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `Payment.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `Payment.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `Payment.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
| `Payment.cardToken` | `String` | Não | Token do cartão utilizado na transação. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        PaymentRequestV2 request = new PaymentRequestV2();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.startPaymentV2(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(PaymentV2 payment) {
        Log.i(TAG, payment.toString());
    }
}
```


### `confirmPayment()`

Este método deve ser chamado para confirmar uma transação que o terminal conseguiu processar completamente a perna de autorização enviada pelo Autorizador.

Este método **não** deve ser chamado para uma transação já confirmada, ou seja, em que já se executou o método **confirmPayment()** anteriormente.

Este método **não** deve ser chamado para uma transação já desfeita, ou seja, em que já se executou o método **cancelPayment()** anteriormente.

Este método **não** deve ser chamado para uma transação que foi negada pelo Autorizador, ou seja, a transação precisa ter sido autorizada pelo Autorizador.


Após a execução desta confirmação, a transação só poderá ser cancelada através de uma operação de estorno (o estorno é a operação executada pelo menu CANCELAMENTO do terminal). 

Caso o App consumidor desta API tenha finalizado o seu processo de negócio com êxito, porém não tenha chamado o método **confirmPayment()**, a transação permanecerá com a seguintes situações:
    Situação PayStore       = "Pendente".    
    Resolução no Adquirente = "Pendente".

Como resultado, poderemos ter uma inconsistência transacional, visto que, na virada do dia, algumas redes adquirentes confirmam automaticamente as transações que não receberam a perna de confirmação. Outras redes adquirentes trabalham apenas com duas pernas, sem a necessidade de perna de confirmação. Neste caso, se houver algum problema na conclusão da transação no lado do terminal, é imperativo que a solução de captura execute o método **cancelPayment()**, a fim de desfazer a transação no adquirente e evitar cobrança para o cliente portador do cartão. 

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `paymentId` | `String` | Sim | Identificador da transação que será confirmada para a aplicação de pagamentos. | 
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        PaymentRequest request = new PaymentRequest();
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
    public void onSuccess(Payment payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.confirmPayment(payment.getPaymentId(),
                                         new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(Payment payment) {
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

Este método deve ser sempre chamado para desfazer uma transação que o terminal não conseguiu receber/processar/imprimir a perna de autorização enviada pelo Autorizador.

Este método **não** deve ser chamado para uma transação já confirmada, ou seja, em que já se executou o método **confirmPayment()**.

Este método **não** deve ser chamado para desfazer uma transação já desfeita.

Este método **não** deve ser chamado para uma transação que foi negada pelo Autorizador.

Este método **não** é um estorno (o estorno é a operação executada pelo menu CANCELAMENTO do terminal). O estorno é executado em transações que foram concluídas com êxito, ou seja, estão confirmadas.

Após a execução do desfazimento, **cancelPayment()**, a transação não poderá ser mais confirmada pela aplicação do terminal, ou seja, não se pode mais executar o método **confirmPayment()**.

Caso o App consumidor desta API não tenha finalizado o seu processo de negócio com êxito, é imprescindível a chamada do método **cancelPayment()**. A consequência de não cancelar uma transação que não teve seu processo de negócio concluído é semelhante à consequência de não confirmar. Porém, nesse caso, com um agravante, pois provavelmente o cliente não levará o produto/serviço associado à transação financeira, ou uma nova tentativa de venda poderá ser feita, resultando em uma cobrança em duplicidade para o cliente portador do cartão.

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `paymentId` | `String` | Sim | Identificador da transação que será desfeita para a aplicação de pagamentos. |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        PaymentRequest request = new PaymentRequest();
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
    public void onSuccess(Payment payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.cancelPayment(payment.getPaymentId(), 
                                        new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(Payment payment) {
                    Log.i(TAG, payment.toString());
                }
            
            
            });
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }
}
```

### `reversePayment()` **( DEPRECATED : Utilizar reversePaymentV2 )**

Este método deve ser chamado quando se deseja fazer uma solicitação de estorno de pagamento. Durante sua execução, os dados do estorno serão validados, informações adicionais serão solicitadas ao operador (e.g. cartão) e a autorização junto à adquirente será feita.

Note que a transação de estorno não possui confirmação, mas apenas desfazimento. Assim, a confirmação ocorrerá naturalmente com o não envio do desfazimento, a depender do comportamento de cada adquirente.

Também a depender do comportamento de cada adquirente, é possível que não haja desfazimento para a transação de estorno para uma determinada adquirente. Neste caso, estornos aprovados retornarão o valor _false_ no campo "ReversePayment.cancelable". Além disto, caso seja chamado o método `cancelReversePayment()`, um erro específico será retornado informando que não é possível executar tal operação (vide [Códigos de Resposta](#codigos-de-resposta)).

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `ReversePaymentRequest` | Sim | Objeto de transferência de dados que conterá as informações da requisição do estorno do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `ReversePaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de estorno.   |
    
**Detalhe dos parâmetros**  
  
_request (ReversePaymentRequest)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor da transação a ser estornada. Caso não seja preenchido (null), a interface solicitará o valor do operador. Esta informação é utilizada para validar a integridade da transação que está sendo estornada. |
| `paymentId` | `String` | Sim | Identificador da transação que será estornada para a aplicação de pagamentos. |
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de estorno. Deve ser o mesmo valor enviado na transação de pagamento. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 
| `showReceiptView` | `IGNORADO` | Não | A Solução sempre irá imprimir o comprovante.  

_callback (ReversePaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `ReversePayment.paymentId` | `String` | Sim | Identificador da transação de estorno para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `ReversePayment.acquirerId` | `String` | Sim | Identificador da transação de estorno para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do estorno com a transação integrada. |
| `ReversePayment.cancelable` | `Boolean` | Sim | _True_, caso esta transação possa ser desfeita. _False_ caso contrário. |
| `ReversePayment.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `ReversePayment.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `ReversePayment.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `ReversePayment.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `ReversePayment.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        ReversePaymentRequest request = new ReversePaymentRequest();
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
    public void onSuccess(ReversePayment payment) {
        Log.i(TAG, payment.toString());
    }
}
```

### `reversePaymentV2()`

Este método deve ser chamado quando se deseja fazer uma solicitação de estorno de pagamento. Durante sua execução, os dados do estorno serão validados, informações adicionais serão solicitadas ao operador (e.g. cartão) e a autorização junto à adquirente será feita.

Note que a transação de estorno não possui confirmação, mas apenas desfazimento. Assim, a confirmação ocorrerá naturalmente com o não envio do desfazimento, a depender do comportamento de cada adquirente.

Também a depender do comportamento de cada adquirente, é possível que não haja desfazimento para a transação de estorno para uma determinada adquirente. Neste caso, estornos aprovados retornarão o valor _false_ no campo "ReversePayment.cancelable". Além disto, caso seja chamado o método `cancelReversePayment()`, um erro específico será retornado informando que não é possível executar tal operação (vide [Códigos de Resposta](#codigos-de-resposta)).

**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `request` | `ReversePaymentRequestV2` | Sim | Objeto de transferência de dados que conterá as informações da requisição do estorno do pagamento. Note que nem todos os parâmetros são obrigatórios.  |
| `callback` | `ReversePaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro do processo de estorno.   |
    
**Detalhe dos parâmetros**  
  
_request (ReversePaymentRequestV2)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `value` | `BigDecimal` | Não | Valor da transação a ser estornada. Caso não seja preenchido (null), a interface solicitará o valor do operador. Esta informação é utilizada para validar a integridade da transação que está sendo estornada. |
| `paymentId` | `String` | Sim | Identificador da transação que será estornada para a aplicação de pagamentos. |
| `appTransactionId` | `String` | Sim | Identificador da transação integrada para o software que originou a solicitação de estorno. Deve ser o mesmo valor enviado na transação de pagamento. |
| `ApplicationInfo.credentials` | `Credentials` | Sim | Credenciais da aplicação que está solicitando a operação, conforme cadastro na PayStore. Basicamente, trata-se da identificação da aplicação e o token de acesso. | 
| `ApplicationInfo.softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando o pagamento. | 
| `showReceiptView` | `IGNORADO` | Não | A Solução irá utilizar o valor dos parâmetros `printMerchantReceipt` e `printCustomerReceipt` para executar a impressão. |
| `printMerchantReceipt` | `Boolean` | Não | Indica se o comprovante do estabelecimento deve ser impresso. O valor padrão é _false_. | 
| `printCustomerReceipt` | `Boolean` | Não | Indica se o comprovante do cliente deve ser impresso. O valor padrão é _false_. |   

_callback (ReversePaymentCallback)_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
| `ReversePayment.paymentId` | `String` | Sim | Identificador da transação de estorno para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `ReversePayment.acquirerId` | `String` | Sim | Identificador da transação de estorno para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do estorno com a transação integrada. |
| `ReversePayment.cancelable` | `Boolean` | Sim | _True_, caso esta transação possa ser desfeita. _False_ caso contrário. |
| `ReversePayment.acquirerResponseCode` | `String` | Sim | Código de resposta da adquirente. |
| `ReversePayment.acquirerResponseDate` | `String` | Sim | Data/hora retornada pela adquirente. |
| `ReversePayment.acquirerAuthorizationNumber` | `String` | Sim | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `ReversePayment.Receipt.clientVia` | `String` | Não | Conteúdo do comprovante - via do cliente. |
| `ReversePayment.Receipt.merchantVia` | `String` | Não | Conteúdo do comprovante - via do estabelecimento. |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta) |
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        ReversePaymentRequestV2 request = new ReversePaymentRequestV2();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        request.setPaymentId("999999");
        request.setPrintCustomerReceipt(true);
        request.setPrintMerchantReceipt(true);
        
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        request.setApplicationInfo(appInfo);

        try {
            paymentClient.reversePaymentV2(request, this);
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }

    @Override
    public void onError(String errorMessage) {
        Log.e(TAG, errorMessage);
    }

    @Override
    public void onSuccess(ReversePayment payment) {
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
| `paymentId` | `String` | Sim | Identificador da transação que será desfeita para a aplicação de pagamentos. |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros**  
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorData.acquirerResponseCode` | `String` | Não | Código de resposta para o erro ocorrido retornado pela adquirente. Note que este erro só será retornado se a transação for não autorizada pela adquirente. |
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa da não autorização. Caso a transação tenha sido negada pela adquirente, conterá a mensagem retornada pela adquirente. |

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
        ReversePaymentRequest request = new ReversePaymentRequest();
        request.setValue(new BigDecimal(50));
        request.setAppTransactionId("123456");
        request.setPaymentId("999999");
        
        ApplicationInfo appInfo = new ApplicationInfo();
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
    public void onSuccess(ReversePayment payment) {
        Log.i(TAG, payment.toString());

        try {
            paymentClient.cancelReversePayment(payment.getPaymentId(),
                                               new PaymentCallback() {
            
                @Override
                public void onError(String errorMessage) {
                    Log.e(TAG, errorMessage);
                }
            
                @Override
                public void onSuccess(Payment payment) {
                    Log.i(TAG, payment.toString());
                }
            
            
            });
        } catch (ClientException e) {
            Log.e(TAG, "Error starting payment", e);
        }
    }
}
```

### `setTheme()`

Este método deve ser chamado para definir um tema com padrões de cores para as telas de captura na aplicação de Pagamentos.


**Parâmetros**

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `theme` | `String` | Sim | Nome do tema que será definido (_case-sensitive_). |
| `callback` | `PaymentCallback` | Sim | Interface que será executada para notificações de sucesso ou erro.   |
    
**Detalhe dos parâmetros** 

_theme_

* `RedTheme`
* `PinkTheme`
* `PurpleTheme`
* `DeepPurpleTheme`
* `IndigoTheme`
* `BlueTheme`
* `LightBlueTheme`
* `CyanTheme`
* `TealTheme`
* `GreenTheme`
* `LightGreenTheme`
* `LimeTheme`
* `YellowTheme`
* `AmberTheme`
* `OrangeTheme`
* `DeepOrangeTheme`
* `BrownTheme`
* `GreyTheme`
* `BlueGreyTheme`
  
_callback_

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| Método para notificação em caso de sucesso |
|||||
| **`onError`** ||| Método para notificação em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | Código de resposta para o erro ocorrido. Vide [Códigos de Resposta](#codigos-de-resposta)|
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa do erro. |

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
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        try {
            paymentClient.setTheme("GreyTheme", this);
        } catch (ClientException e) {
            Log.e(TAG, "Error setting theme", e);
        }
    }

    @Override
    public void onError(Object data) {
        Log.e(TAG, "Error: " + errorData.getResponseMessage());
    }

    @Override
    public void onSuccess(Object data) {
        Log.i(TAG, "Success!");
    }
}
```

### `startExtraction()`

Esse mÃ©todo deve ser chamado para fazer a extraÃ§Ã£o dos dados do Payments e das adquirentes sendo utilizadas e enviÃ¡-los para a AWS (Amazon Web Services).

**ParÃ¢metros**

| Nome | Tipo | ObrigatÃ³rio | DescriÃ§Ã£o |
| -------- | -------- | -------- | -------- |
| `callback` | `PaymentCallback` | Sim | Interface que serÃ¡ executada para notificaÃ§Ãµes de sucesso ou erro.   |

**Detalhe dos parÃ¢metros** 

_callback_

| Nome | Tipo | ObrigatÃ³rio | DescriÃ§Ã£o |
| -------- | -------- | -------- | -------- |
| **`onSuccess`** ||| MÃ©todo para notificaÃ§Ã£o em caso de sucesso |
|||||
| **`onError`** ||| MÃ©todo para notificaÃ§Ã£o em caso de erro. |
| `ErrorData.paymentsResponseCode` | `String` | Sim | CÃ³digo de resposta para o erro ocorrido. Vide [CÃ³digos de Resposta](#codigos-de-resposta)|
| `ErrorData.responseMessage` | `String` | Sim | Mensagem descritiva da causa do erro. |

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
        ApplicationInfo appInfo = new ApplciationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");
        
        try {
            paymentClient.startExtraction(this);
        } catch (ClientException e) {
            Log.e(TAG, "Error while uploading data.", e);
        }
    }

    @Override
    public void onError(Object data) {
        Log.e(TAG, "Error: " + errorData.getResponseMessage());
    }

    @Override
    public void onSuccess(Object data) {
        Log.i(TAG, "Success!");
    }
}
```

# Integração com Aplicação de Pagamentos via _Content Povider_

A integração via _Content Provider_ visa possibilitar que outros aplicativos possam consultar informações a respeito de pagamentos efetuados, sendo possível realizar filtros e obter diversos dados dos pagamentos, inclusive sua situação.

Só será permitido listar pagamentos feitos pela pela própria aplicação que está realizando a consulta.

Declare essa permissão no AndroidManifest.xml do seu Aplicativo para ter acesso ao _Content Provider_.

`<uses-permission android:name="br.com.phoebus.android.payments.provider.READ_PERMISSION" />`

## content://br.com.phoebus.android.payments.provider/payments
URI para obtenção de informações de pagamentos.

### Filtros

| Nome | Tipo | Obrigatório | Descrição |
| -------- | -------- | -------- | -------- |
| `status` | `PaymentStatus` | Não | Filtra os pagamentos cuja situação está na lista passada (aceita mais de um valor). |
| `startDate` | `Date` | Não | Filtra os pagamentos cuja data seja maior ou igual ao valor passado. |
| `finishDate` | `Date` | Não | Filtra os pagamentos cuja data seja menor ou igual ao valor passado. |
| `startValue` | `BigDecimal` | Não | Filtra os pagamentos cujo valor seja maior ou igual ao valor passado. |
| `finishValue` | `BigDecimal` | Não | Filtra os pagamentos cujo valor seja menor ou igual ao valor passado. |
| `paymentId` | `String` | Não | Filtra os pagamentos cujo identificador da transação para a aplicação de pagamentos seja o valor passado. |
| `lastDigits` | `String` | Não | Filtra os pagamentos cujos últimos 4 dígitos do PAN do cartão usado na transação seja igual ao valor passado. |
| `applicationId` | `Credentials` | Sim | Identificação da aplicação que está realizando a consulta. | 
| `secretToken` | `Credentials` | Sim | Token de acesso da aplicação que está realizando a consulta. | 
| `softwareVersion` | `String` | Sim | Versão da aplicação que está solicitando a consulta. | 

### Retorno

| Nome | Tipo |  Descrição |
| -------- | -------- | -------- |
| `id` | `String` | Identificador da transação para a aplicação de pagamentos. Esta é a informação a ser usada para a confirmação e desfazimento. |
| `value` | `BigDecimal` | Valor do pagamento. Este é o valor que foi aprovado pela adquirente. Deve ser validado sempre na resposta, ainda que tenha sido passado como parâmetro, pois há adquirentes que, para algumas situações, aprovam valores diferentes dos solicitados. |
| `paymentType` | `PaymentType` | Tipo de pagamento (Débito, Crédito, Voucher, etc.) usado no pagamento. |
| `installments` | `Integer` | Quantidade de parcelas do pagamento. |
| `acquirerName` | `String` | Adquirente que autorizou o pagamento. |
| `cardBrand` | `String` | Bandeira do cartão usado no pagamento. |
| `cardBin` | `String` | Bin do cartão usado no pagamento. |
| `cardPanLast4Digits` | `String` | Últimos 4 dígitos do PAN do cartão usado na transação. |
| `captureType` | `CaptureType` | Forma de captura do cartão usado na transação. |
| `status` | `PaymentStatus` | Situação do pagamento. |
| `date` | `Date` | Data/hora do pagamento para a aplicação de pagamentos. |
| `acquirerId` | `String` | Identificador da transação para a adquirente. Identificador que consta no arquivo que a adquirente fornece, de forma que viabilize a conciliação do pagamento com a transação integrada. |
| `acquirerResponseCode` | `String` | Código de resposta da adquirente. |
| `acquirerResponseDate` | `String` | Data/hora retornada pela adquirente. |
| `acquirerAuthorizationNumber` | `String` | Número da autorização fornecido pela adquirente (que consta no comprovante do portador do cartão). |
| `receiptClient` | `String` | Conteúdo do comprovante - via do cliente. |
| `receiptMerchant` | `String` | Conteúdo do comprovante - via do estabelecimento. 

**Atenção**: O objeto retornado pode ser preenchido seletivamente. Para isso, deve ser passado um array de Strings com as colunas desejadas para o método query() do Content Resolver. Caso utilize nossa API de acesso ao provider, pode utilizar o método setColumns() do PaymentProviderRequest.

##### Exemplo

```java
public class MyActivity extends Activity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.your_actitivy);

        //definindo as credenciais
        ApplicationInfo appInfo = new ApplicationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");

        //criando objeto de request para o payment content provider
        PaymentProviderRequest request = createRequest(appInfo);

        //selecionando propriedades retornadas
        String[] columns = new String[]{
            PaymentContract.column.ID,
            PaymentContract.column.VALUE,
            PaymentContract.column.PAYMENT_STATUS,
            PaymentContract.column.PAYMENT_DATE,
            PaymentContract.column.CARD_BRAND,
        };

        //solicitando a lista de pagamentos
        Cursor cursor = getContext().getContentResolver().query(request.getUriBuilder().build(), columns, null, null, null);

        //parse de Cursor para List<Payments>
        List<Payment> paymentsList = Payment.fromCursor(cursor);
    }

    private PaymentProviderRequest createRequest(ApplicationInfo appInfo){
        PaymentProviderRequest paymentRequest = new PaymentProviderRequest(appInfo);

        //filtrando pelo paymentId
        paymentRequest.setPaymentId("000000000");

        //filtrando pelo por periodo
        //utilizar o formato dd/MM/yyyy
        paymentRequest.setStartDate("01/01/2017");
        paymentRequest.setFinishDate("01/06/2017");

        //filtrando por faixa de valores de pagamento
        paymentRequest.setStartValue(new BigDecimal("0,00"));
        paymentRequest.setFinishValue(new BigDecimal("1000,00"));

        //filtrando pelos 4 últimos dígitos do cartão
        paymentRequest.setLastDigits("4456");

        //filtrando pelos 4 últimos dígitos do cartão
        List<PaymentStatus> status = Arrays.asList(new PaymentStatus[]{
            PaymentStatus.PENDING,
            PaymentStatus.CONFIRMED,
            PaymentStatus.CANCELLED,
            PaymentStatus.REVERSED
        });

        return paymentRequest;
    }

}
```

## API

Para facilitar o uso, é disponibilizada uma API de acesso ao provider, com o uso das classes:

- `PaymentProviderRequest`
- `PaymentProviderApi`
- `PaymentContract`

##### Exemplo utilizando PaymentProviderApi

```java
public class MyActivity extends Activity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.your_actitivy);

        //definindo as credenciais
        ApplicationInfo appInfo = new ApplicationInfo();
        appInfo.setCredentials(new Credentials("demo-app", "TOKEN-KEY-DEMO"));
        appInfo.setSoftwareVersion("1.0.0.0");

        //criando objeto de request para o payment content provider
        PaymentProviderRequest request = createRequest(appInfo);

        //selecionando propriedades retornadas
        request.setColumns(new String[]{
            PaymentContract.column.ID,
            PaymentContract.column.VALUE,
            PaymentContract.column.PAYMENT_STATUS,
            PaymentContract.column.PAYMENT_DATE,
            PaymentContract.column.CARD_BRAND,
        });

        //parse de Cursor para List<Payments>
        List<Payment> paymentsList = PaymentProviderApi.create(this).findAll(request);
    }

    private PaymentProviderRequest createRequest(ApplicationInfo appInfo){
        PaymentProviderRequest paymentRequest = new PaymentProviderRequest(appInfo);

        //filtrando pelo por periodo
        //utilizar o formato dd/MM/yyyy
        paymentRequest.setStartDate("01/01/2017");
        paymentRequest.setFinishDate("01/06/2017");

        //filtrando pelos 4 últimos dígitos do cartão
        List<PaymentStatus> status = Arrays.asList(new PaymentStatus[]{
            PaymentStatus.CONFIRMED
        });

        return paymentRequest;
    }

}
```

# Integração com Aplicação de Pagamentos via _Broadcast_

Diferente das outras integrações, nesta, outras aplicações podem receber notificações de que um pagamento ou um estorno foi efetuado. Esta informação é enviada pela aplicação de pagamentos via [_Broadcasts_](https://developer.android.com/guide/components/broadcasts.html) que podem ser recebidos por quaisquer aplicações que tenham interesse em saber da ocorrência de pagamentos ou estornos.

## _Actions_

| Action |  Extra |
| -------- | -------- |
| `Intents.action.ACTION_AFTER_PAYMENT` (`br.com.phoebus.android.payments.AFTER_PAYMENT_FINISHED`) |  `Intents.extra.EXTRA_PAYMENT_RETURN`: `Payment` (vide [startPayment()](#startpayment) ou documentação da classe.) |
| `Intents.action.ACTION_AFTER_REVERSAL` (`br.com.phoebus.android.payments.AFTER_PAYMENT_REVERSAL_FINISHED`) |  `Intents.extra.EXTRA_PAYMENT_RETURN`: `ReversePayment` (vide [reversePayment()](#reversepayment) ou documentação da classe.) |

## Exemplo

```java
public class MyBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {

        if (intent.getAction().equals(Intents.action.ACTION_AFTER_PAYMENT)) {
            Payment payment = DataUtils.fromBundle(Payment.class, intent.getExtras(), Intents.extra.EXTRA_PAYMENT_RETURN);
            
            // Do something!
            
        } else if (intent.getAction().equals(Intents.action.ACTION_AFTER_REVERSAL)) {
            ReversePayment reversePayment = DataUtils.fromBundle(ReversePayment.class, intent.getExtras(), Intents.extra.EXTRA_PAYMENT_RETURN);

            // Do something!

        }

    }
    
}
```



# Códigos de Resposta

| Código | Descrição | Operações |
| -------| --------- | --------- |
| 01     | Transação negada pela adquirente. | `startPayment` e `startPaymentV2` |
| 02     | Transação negada pelo cartão. | `startPayment` e `startPaymentV2`|
| 03     | Operação cancelada pelo operador. | `startPayment`, `startPaymentV2` , `reversePayment` e `reversePaymentV2` |
| 04     | Pagamento não encontrado. | `confirmPayment`, `cancelPayment`, `reversePayment`, `reversePaymentV2` e `cancelReversePayment` |
| 05     | Problerma na comunicação com o aplicativo de pagamento. | Todas |
| 06     | Operação não disponível na adquirente. | `cancelReversePayment` |
| 07     | Problema de comunicação com a adquirente. | Todas |
| 08     | Credenciais Inválidas. | `startPayment`, `startPaymentV2`, `reversePayment` e `reversePaymentV2`|
| 09     | Aplicativo de Pagamentos não possui permissões para continuar . | `startPayment`,`startPaymentV2`, `reversePayment` e `reversePaymentV2` |
| 10     | Terminal Bloqueado. | `startPayment`, `startPaymentV2`, `reversePayment` e `reversePaymentV2`|
| 11     | Pagamento bloqueado pois existe transação pendente. | `startPayment`, `startPaymentV2`, `reversePayment` e `reversePaymentV2`|
| 12     | Tema inválido                                       |  `setTheme` |
| 99     | Problema Interno. | Todas |
