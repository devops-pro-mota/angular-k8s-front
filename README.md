Podemos seguir algumas praticas recomendadas. Como Angular é uma aplicação de frontend que é servida como arquivos estáticos, a melhor forma de carregar variáveis de configuração, como URLs de APIs ou dados sensíveis, é através do uso de um arquivo JSON que é gerado ou modificado durante o processo de build ou deploy.

Aqui está uma abordagem passo a passo:

1. **Criar um arquivo JSON de configuração**: Crie um arquivo, por exemplo, `config.json`, que incluirá as variáveis que sua aplicação Angular precisa acessar. O arquivo pode ter a seguinte estrutura:

   ```json
   {
       "apiUrl": "https://api.parceiro-integracao-mota.com",
       "outroDado": "valor"
   }
   ```
Essa parte é minha - DevOps

2. **Utilizar um ConfigMap no Kubernetes**: Crie um ConfigMap que inclua esse arquivo JSON. Você pode fazer isso com um comando `kubectl` ou definindo um ConfigMap em um arquivo YAML:

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
       name: minha-config-map
   data:
       config.json: |
           {
               "apiUrl": "https://api.parceiro-integracao-mota.com",
               "outroDado": "valor"
           }
   ```
Essa parte é minha - DevOps
3. **Montar o ConfigMap como um volume**: No seu arquivo de deployment do Kubernetes, monte o ConfigMap como um volume no contêiner onde sua aplicação Angular está sendo executada.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
       name: minha-aplicacao
   spec:
       replicas: 1
       template:
           spec:
               containers:
               - name: minha-aplicacao
                 image: minha-imagem-angular
                 volumeMounts:
                 - name: config-volume
                   mountPath: /usr/share/nginx/html/assets/config.json
                   subPath: config.json
               volumes:
               - name: config-volume
                 configMap:
                   name: minha-config-map
   ```

4. **Carregar a configuração no Angular**: Dentro da sua aplicação Angular, você pode fazer uma requisição HTTP para carregar o arquivo JSON de configuração ao inicializar a aplicação. Um exemplo de serviço Angular para carregar configurações poderia ser:

   ```typescript
   import { Injectable } from '@angular/core';
   import { HttpClient } from '@angular/common/http';

   @Injectable({
     providedIn: 'root'
   })
   export class ConfigService {
     private config: any;

     constructor(private http: HttpClient) {}

     loadConfig() {
       return this.http.get('/assets/config.json')
         .toPromise()
         .then(data => {
           this.config = data;
         });
     }

     get apiUrl() {
       return this.config.apiUrl;
     }
   }
   ```

5. **Inicialização da configuração**: Garanta que a configuração seja carregada antes da inicialização do seu aplicativo. Você pode usar um `APP_INITIALIZER` para garantir que o serviço de configuração seja carregado antes que a aplicação Angular seja inicializada.

   ```typescript
   import { NgModule, APP_INITIALIZER } from '@angular/core';
   import { BrowserModule } from '@angular/platform-browser';
   import { HttpClientModule } from '@angular/common/http';
   import { ConfigService } from './config.service';

   export function initializeApp(configService: ConfigService) {
     return (): Promise<any> => {
       return configService.loadConfig();
     }
   }

   @NgModule({
     imports: [BrowserModule, HttpClientModule],
     providers: [
       {
         provide: APP_INITIALIZER,
         useFactory: initializeApp,
         deps: [ConfigService],
         multi: true
       }
     ],
     bootstrap: [/* seu componente principa aqui é o front l */]
   })
   export class AppModule {}
   ```

Essa abordagem garante que sua aplicação Angular tenha acesso às configurações que você deseja importar do Kubernetes de forma segura e flexível. Além disso, evita o hardcoding de valores sensíveis dentro do código-fonte da sua aplicação.
