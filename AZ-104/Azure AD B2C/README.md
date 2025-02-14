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
    É a definição de uma regra de validação para campos ClaymType, seria uma "função" que valida uma regra especifica.

    Essa seria a definição das "funções" e das valiidações:
    ```xml
    <Predicates>
        <Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
            <Parameters>
                <Parameter Id="Minimum">8</Parameter>
                <Parameter Id="Maximum">64</Parameter>
            </Parameters>
        </Predicate>

        <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
            <Parameters>
                <Parameter Id="CharacterSet">a-z</Parameter>
            </Parameters>
        </Predicate>

        <Predicate Id="DisallowedWhitespace" Method="MatchesRegex" HelpText="The password must not begin or end with a whitespace character.">
            <Parameters>
                <Parameter Id="RegularExpression">(^\S.*\S$)|(^\S+$)|(^$)</Parameter>
            </Parameters>
        </Predicate>
    </Predicates>
    ```
   
4. [PredicateValidations](https://learn.microsoft.com/en-us/azure/active-directory-b2c/predicates#predicatevalidations)
   É como um middleware que agrupa as validações predicate.

   No Exemplo abaixo temos a definição de dois validatores.
    O `StrongPassword` define uma serie de chamadas encadeadas a predicates declarados.
   ```xml
     <PredicateValidation Id="StrongPassword">
        <PredicateGroups>
            <PredicateGroup Id="DisallowedWhitespaceGroup">
                <PredicateReferences>
                    <PredicateReference Id="DisallowedWhitespace" />
                </PredicateReferences>
            </PredicateGroup>
            <PredicateGroup Id="AllowedAADCharactersGroup">
                <PredicateReferences>
                    <PredicateReference Id="AllowedAADCharacters" />
                </PredicateReferences>
            </PredicateGroup>
            <PredicateGroup Id="LengthGroup">
                <PredicateReferences>
                    <PredicateReference Id="IsLengthBetween8And64" />
                </PredicateReferences>
            </PredicateGroup>
            <PredicateGroup Id="CharacterClasses">
                <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
                <PredicateReferences MatchAtLeast="3">
                    <PredicateReference Id="Lowercase" />
                    <PredicateReference Id="Uppercase" />
                    <PredicateReference Id="Number" />
                    <PredicateReference Id="Symbol" />
                </PredicateReferences>
            </PredicateGroup>
        </PredicateGroups>
    </PredicateValidation>

    <PredicateValidation Id="CustomPassword">
        <PredicateGroups>
            <PredicateGroup Id="DisallowedWhitespaceGroup">
                <PredicateReferences>
                    <PredicateReference Id="DisallowedWhitespace" />
                </PredicateReferences>
            </PredicateGroup>
            <PredicateGroup Id="AllowedAADCharactersGroup">
                <PredicateReferences>
                    <PredicateReference Id="AllowedAADCharacters" />
                </PredicateReferences>
            </PredicateGroup>
        </PredicateGroups>
    </PredicateValidation>
   ```

    **Exemplo de aplicação de um PredicateValidation em um ClaymType**
    ```xml
    <ClaimType Id="password">
        <DisplayName>Password</DisplayName>
        <DataType>string</DataType>
        <AdminHelpText>Enter password</AdminHelpText>
        <UserHelpText>Enter password</UserHelpText>
        <UserInputType>Password</UserInputType>
        <!-- Aqui vemos a definicação de uma validação StrongPassword -->
        <PredicateValidationReference Id="StrongPassword" />
    </ClaimType>
    ```

5. [ContentDefinitions](https://learn.microsoft.com/en-us/azure/active-directory-b2c/contentdefinitions#contentdefinition)
    Define estilos personalizados que poder ser utilizados para estilizar technical profile.

    É como a declaração de estruturas html que pode ser utilizadas para alterar como é exibidos partes do B2C.
    Por Exemplo se definirmos `api.localaccountsignup` podemos usar em um technical profile.
    ````xml
    <ContentDefinition Id="api.localaccountsignup">
        <LoadUri>~/tenant/default/selfAsserted.cshtml</LoadUri>
        <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
        <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:1.1.0</DataUri>
        <Metadata>
            <Item Key="DisplayName">Local account sign up page</Item>
        </Metadata>
        <LocalizedResourcesReferences MergeBehavior="Prepend">
            <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
            <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
            ...
    ```

    Technical profile example:
    ```xml
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
        <DisplayName>Email signup</DisplayName>
        <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Metadata>
            <!-- Definição do ContentDefinition -->
            <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
            ...
        </Metadata>
        ...
    ```
6. [Localization](https://learn.microsoft.com/en-us/azure/active-directory-b2c/localization)
7. [Display Controls](https://learn.microsoft.com/en-us/azure/active-directory-b2c/display-controls)