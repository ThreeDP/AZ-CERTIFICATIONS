# Azure AD B2C

Esse tópico tem como foco descrever as meu fluxo de aprendizado para o Azure AD B2C.

## - Oque são Policy Keys e qual sua funcionalidade
    - o que é um `TokenSigningKeyContainer`
    - o que é um `TokenEncryptionKeyContainer`

## Oque é o Identity Experience Framework


## Tipos de Autenticação
### - Oque é OAuth2
### - Oque é OpenID Connect
### - Oque é JWTs

## Auteticação em detalhes 
### - Oque é Scopes
### - O que São Claims
### - Oque são Tokens
### - Diferença entre um Identity provider em um identity service

# Custom Policy

## Arquivo Base

### TrustFrameworkPolicy


### [BuildingBlocks](https://learn.microsoft.com/en-us/azure/active-directory-b2c/buildingblocks)

1. [ClaimsSchema](https://learn.microsoft.com/en-us/azure/active-directory-b2c/claimsschema):
os ClaimTypes são responsáveis por declarar, validar, e manipular a visualização das claims que serão geradas. São Declarações de Tipos.

- `DefaultPartnerClaimTypes:`
    > é utilizado para definir os tipos de reivindicação padrão que um parceiro usará para um protocolo específico em um sistema de identidade baseado em reivindicações. Isso permite uma maneira padronizada de lidar com reivindicações, garantindo que as informações necessárias sejam comunicadas de forma consistente entre as partes. 
    > eles podem ser substituídos pelo atributo `PartnerClaimType` nos elementos `InputClaim` ou `OutputClaim` \
    > \
    > `Exemplo:`
    > Nesse ClaimType abaixo definimos como o *displayName* será interpretado em diferentes tipos de protocolos de auth.
    > ```
    > <ClaimType Id="displayName">
    >     <DisplayName>Display Name</DisplayName>
    >     <DataType>string</DataType>
    >     <DefaultPartnerClaimTypes>
    >       <Protocol Name="OAuth2" PartnerClaimType="unique_name" />
    >       <Protocol Name="OpenIdConnect" PartnerClaimType="name" />
    >       <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" />
    >     </DefaultPartnerClaimTypes>
    >     <UserHelpText>Your display name.</UserHelpText>
    >     <UserInputType>TextBox</UserInputType>
    >   </ClaimType>
    > ```

2. [ClaimsTransformations](https://learn.microsoft.com/en-us/azure/active-directory-b2c/claimstransformations)
   São funções que serão executadas durante uma UserJourney para obter / transformar as claims necessarias.

    - [TransformationMethod](https://learn.microsoft.com/en-us/azure/active-directory-b2c/claimstransformations#claims-transformations-reference): É a "função" que será executada quando essa ClaimTransformation for chamada.
    - `Id`: é o nome da claim tranformation
    - Blocos do ClaimsTranformation `InputClaims`, `InputParameters` e `OutputClaims`.
      - `InputClaims:` Define os tipos de Claim que será aceitas como paramentos, devem estar declaradas no ClaimsSchema.
      - `InputParameters:`  Parâmetros adicionais que influenciam a transformação, mas que não são Claims. São parâmetros adicionais fornecidos como entrada para a transformação de Claims. Esses parâmetros podem ser usados para modificar ou influenciar o comportamento da transformação.
      - `OutputClaims:` Claims que iram conter o resultado da execução do método, devem estar declaradas no ClaimsSchema.
    > Exemplo de claim Transformation
    > ```xml
    > 
    > <BuildingBlocks>
    >     <ClaimsSchema>
    >         <ClaimType Id="TOSVersionChanged">
    >         <DisplayName>Indicates if the TOS version accepted by the end user is equal to the current version</DisplayName>
    >         <DataType>boolean</DataType>
    >         </ClaimType>
    >         <ClaimType Id="TOSVersion">
    >         <DisplayName>TOS version</DisplayName>
    >         <DataType>string</DataType>
    >         </ClaimType>
    >         <ClaimType Id="LastTOSAcceptedVersion">
    >         <DisplayName>TOS version accepted by the end user</DisplayName>
    >         <DataType>string</DataType>
    >         </ClaimType>
    >     </ClaimsSchema>
    > 
    >     <ClaimsTransformations>
    >         <ClaimsTransformation Id="HasTOSVersionChanged" TransformationMethod="CompareClaims">
    >         <InputClaims>
    >             <InputClaim ClaimTypeReferenceId="TOSVersion" TransformationClaimType="inputClaim1" />
    >             <InputClaim ClaimTypeReferenceId="LastTOSAcceptedVersion" TransformationClaimType="inputClaim2" />
    >         </InputClaims>
    >         <InputParameters>
    >             <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
    >         </InputParameters>
    >         <OutputClaims>
    >             <OutputClaim ClaimTypeReferenceId="TOSVersionChanged" TransformationClaimType="outputClaim" />
    >         </OutputClaims>
    >         </ClaimsTransformation>
    >     </ClaimsTransformations>
    > </BuildingBlocks>
    > ```
    > Esse bloco seria algo como isso no C#
    > ```cs
    > public TOSVersionChanged HasTOSVersionChanged(TOSVersion inputClaim1, LastTOSAcceptedVersion inputClaim2)
    > {
    >    return CompareClaims(inputClaim1, inputClaim2, "NOT EQUAL", ignoreCase=true);
    > }
    > ```
3. [Predicates](https://learn.microsoft.com/en-us/azure/active-directory-b2c/predicates)
   
4. [PredicateValidations](https://learn.microsoft.com/en-us/azure/active-directory-b2c/predicates#predicatevalidations)