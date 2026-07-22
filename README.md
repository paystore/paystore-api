# paystore-api

Biblioteca oficial para integração com o aplicativo de pagamentos **PayStore Payments App**.

A documentação completa da API está disponível [aqui](http://177.69.97.18:6655).

---

## 🔄 Compatibilidade entre payments-api (AAR) e PayStore Payments App

A tabela abaixo apresenta as combinações homologadas entre as versões do **payments-api (AAR)** e o **PayStore Payments App**:

| Versão AAR              | Payments App compatíveis (v2.11.x) | Payments App compatíveis (v2.12.x) |
| ----------------------- | ---------------------------------- | ---------------------------------- |
| `payments-api-2.10.6.8` | v2.10.6.13                         | N/A                                |
| `payments-api-4.0.1.4`  | N/A                                | v2.12.3.10 a v2.12.3.12            |
| `payments-api-4.0.2.8`  | v2.11.7.7 a v2.11.7.14             | v2.12.3.13 a v2.12.3.18            |
| `payments-api-4.1.0.1`  | v2.11.7.15                         | v2.12.3.19 a v2.12.3.20            |
| `payments-api-4.1.0.3`  | v2.11.8                            | v2.12.7                            |
| `payments-api-4.1.0.5`  | v2.11.8                            | v2.12.7                            |
| `payments-api-4.1.0.6`  | v2.11.8 a v2.11.9                  | v2.12.7 a v2.12.8                  |
| `payments-api-4.1.0.7`  | v2.11.8 a v2.11.9                  | v2.12.8 a v2.12.9                  |
| `payments-api-4.1.0.9`  | v2.11.8 a v2.11.9                  | v2.12.9 a v2.12.10                 |


---

## 📌 Política de Retrocompatibilidade

Todas as versões do **payments-api** são desenvolvidas com garantia de retrocompatibilidade.

Isso significa que:

* Não removemos nem alteramos assinaturas de métodos já utilizados por parceiros.
* Integrações existentes continuam funcionando após a atualização do AAR.
* Novas versões adicionam funcionalidades ou correções sem impactar fluxos já implementados.

---

## 🚀 Sobre a Tabela de Compatibilidade

A tabela acima não indica bloqueio técnico entre versões, mas sim:

* As combinações oficialmente homologadas.
* As versões do Payments App que permitem utilizar novas funcionalidades ou correções introduzidas em cada versão do AAR.

Caso o parceiro utilize uma versão mais antiga do Payments App, a integração continuará funcionando normalmente, porém os novos recursos só estarão disponíveis após a atualização para a versão recomendada.
