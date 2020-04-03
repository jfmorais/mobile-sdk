# Guia de Integração com a Mobile Insights Plataform 

Esse manual descreve o processo de integração da MIP-SDK (Mobile Insights Platform) em aplicativos Android. Essa biblioteca permite a coleta de informações anonimizadas de um dispositivo, de forma a fornecer insumos  para o combate a expansão do COVID-19.

Nesse exemplo, utilizaremos o **Android Studio 3.6** em execução no sistema operacional **macOS Mojave**.

## Requisitos Técnicos

A MIP-SDK requer a API Android igual ou superior a 16 (`minSdkVersion 16`).

# Passos para Integração

A MIP-SDK é responsável por coletar, anonimizar, encriptar e enviar os dados do dispositivo. Os itens a seguir são necessários para a correta integração da biblioteca.

## 1. Importação da biblioteca

O pacote entregue é composto por um arquivo com extensão `.aar` (Android Archive Library), que deverá ser adicionado ao projeto.

Para isso, acessar o menu `File -> New Module...`, conforme a figura abaixo.

![New Module](./06.png)

No menu `New Module`, selecionar a opção `Import .JAR/.AAR Package`

![New Module](./07.png)

Em seguida, devemos informar a localização do arquivo **.aar** enviado pela Serasa Experian.

![New Module](./08.png)

Ao preenchermos o caminho (ou clicando no botão `...`), o campo **Subproject name** é automaticamente preenchido pela interface.

O próximo passo é acrescentar a dependência no arquivo de script `build.gradle`. Para isso, clique duas vezes em `Gradle Scripts` na interface e em seguida no arquivo `build.gradle (Module: app)`.

![New Module](./081.png)

![New Module](./082.png)

Com o arquivo `build.gradle` aberto, incluir a dependência do SDK no atributo `dependencies`, inserindo a linha `implementation project(':android-mobile-sdk-release')` conforme figura.

![New Module](./09.png)

## 2. Metadados da MIP-SDK

Para configurar o metadado, adicione a tag `meta-data` abaixo dentro da tag `<application>` do arquivo `AndroidManiferst.xml`.

* `<meta-data android:name="br.com.experian.mip.PARTNER_CODE" android:value="COVID"/>`

## 3. Especificação da finalidade de uso da localização

A Lei Geral de Proteção de Dados Pessoais (LGPD ou LGPDP), Lei nº 13.709/2018, é a legislação brasileira que regula as atividades de tratamento de dados pessoais e que também altera os artigos 7º e 16 do Marco Civil da Internet.

Para garantir o cumprimento da referida lei, o MIP-SDK deve ser notificado do aceite do usuário nos termos de que seus dados serão utilizados para essa finalidade específica. Para isso, utilize o método listado abaixo, disponível na classe  `br.com.experian.android.mobile.sdk.library.core.ExpMobileInsightsPlatform`:

* `userAcceptedDataForGoodAgreement()`

## 4. Versão do documento Termos e Condições

Além da finalidade, também é necessário estabelecer um vínculo entre o aceite da finalidade e os **Termos e Condições** apresentados. Para isso, utilize o método `privacyPolicy(String arg)` da classe `br.com.experian.android.mobile.sdk.library.core.ExpMobileInsightsPlatform`. O argumento pode ser a versão do documento, o hash do documento, a URL onde o documento está disponível ou a íntegra do próprio texto.

## 5. Integração ProGuard

Caso utilize a ferramenta ProGuard, é necessário adicionar a linha abaixo no arquivo de configuração.

```properties
-keep class br.com.experian.android.mobile.sdk.library.core.** { *; }
```

## 6. Requisitar a localização

Para que a MIP SDK colete a localização o aplicativo deve ter acesso a mesma, para requisitar acesso a localização basta incluir as linhas abaixo no aplicativo:

Arquivo `AndroidManifest.xml`

```xml
<manifest ... >
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
</manifest>
```

```java
boolean permissionAccessCoarseLocationApproved =
    ActivityCompat.checkSelfPermission(this, permission.ACCESS_COARSE_LOCATION)
        == PackageManager.PERMISSION_GRANTED;

if (permissionAccessCoarseLocationApproved) {
   boolean backgroundLocationPermissionApproved =
           ActivityCompat.checkSelfPermission(this,
               permission.ACCESS_BACKGROUND_LOCATION)
               == PackageManager.PERMISSION_GRANTED;

   if (backgroundLocationPermissionApproved) {
       // App can access location both in the foreground and in the background.
       // Start your service that doesn't have a foreground service type
       // defined.
   } else {
       // App can only access location in the foreground. Display a dialog
       // warning the user that your app must have all-the-time access to
       // location in order to function properly. Then, request background
       // location.
       ActivityCompat.requestPermissions(this, new String[] {
           Manifest.permission.ACCESS_BACKGROUND_LOCATION},
           your-permission-request-code);
   }
} else {
   // App doesn't have access to the device's location at all. Make full request
   // for permission.
   ActivityCompat.requestPermissions(this, new String[] {
        Manifest.permission.ACCESS_COARSE_LOCATION,
        Manifest.permission.ACCESS_BACKGROUND_LOCATION
        },
        your-permission-request-code);
}
```

## 7. Coleta de Dados

Para sermos capazes de fornecer informações mais úteis, é necessário habilitar dois tipos de coleta: a) em segundo plano (background) e b) instantânea.

A coleta em segundo plano ocorre de maneira transparente para o usuário e utiliza pouquíssimos recursos de dados e bateria do dispositivo. Para agendar a sua execução recorrente, utilize o método `scheduleContinuousCollection()` da classe `br.com.experian.android.mobile.sdk.library.core.ExpMobileInsightsPlatform` na atividade inicial da aplicação.

A coleta instantânea deve ser executada quando a aplicação está em execução pelo usuário. Solicitamos que essa coleta seja feita, se possível, uma vez ao dia.


## 8. Efetuar o agendamento da coleta continua da localização

Para sermos capazes de fornecer informações mais úteis, é necessário habilitar dois tipos de coleta: a) em segundo plano (background) e b) instantânea.

A coleta em segundo plano ocorre de maneira transparente para o usuário e utiliza pouquíssimos recursos de dados e bateria do dispositivo. Para agendar a sua execução recorrente, utilize o método `scheduleContinuousCollection()` da classe `br.com.experian.android.mobile.sdk.library.core.ExpMobileInsightsPlatform` na atividade inicial da aplicação.

## 9. Extração Única da localização

O MIP-SDK tem como característica principal o respeito ao aplicativo dos seus parceiros. Por esse motivo, o processamento e envio dos dados costuma ocorrer em situações ótimas para o aparelho (carregando e conectado em rede Wi-Fi), o que pode ocasionar algum tempo adicional no envio dos dados. Para garantir pelo menos a primeira extração completa dos dados, é recomendável efetuar uma chamada do método `runOneShotCollection()`.

# Exemplo de Código de Integração

```java
package com.example.myapplication;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import br.com.experian.android.mobile.sdk.library.core.CollectStatus;
import br.com.experian.android.mobile.sdk.library.core.ExpMobileInsightsPlatform;

public class MainActivity extends AppCompatActivity {

  private static final String TAG = "My Application";
  private final ExpMobileInsightsPlatform mip =
    ExpMobileInsightsPlatform.getInstance();

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    //Usuario aceitou os Termos e Condicoes
    //de compartilhamento anonimo dos dados
    mip.userAcceptedDataForGoodAgreement();
    mip.privacyPolicy("1.0");

    //agendamento coleta continua
    mip.scheduleContinuousCollection();

    //se primeira execucao do dia
    if (firstExecutionOfDay()) {
      //execucao coleta instantanea
      runOneShotCollection();
    }
  }

  public void runOneShotCollection() {
    AsyncTask<Void, Void, CollectStatus> async =
      new AsyncTask<Void, Void, CollectStatus>() {
      @Override
      protected CollectStatus doInBackground(Void... params) {
        try {
          return mip.runOneShotCollection();

        } catch (Exception e) {
          Log.e(TAG, "Error sending data", e);
        }
        return null;
      }
      @Override
      protected void onPostExecute(final CollectStatus result) {
        Log.d(TAG, "Result [" + 
          (result != null && result == CollectStatus.SUCCESS)
        + "]");
      }
    };
    async.execute();
  }
}
```