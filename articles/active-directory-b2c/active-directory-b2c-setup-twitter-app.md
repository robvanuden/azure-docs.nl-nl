---
title: 'Azure Active Directory B2C: Twitter configuratie | Microsoft Docs'
description: Registreren en aanmelden gebruikers met Twitter-accounts in uw toepassingen die zijn beveiligd met Azure Active Directory B2C bieden.
services: active-directory-b2c
documentationcenter: ''
author: davidmu1
manager: mtillman
editor: ''
ms.service: active-directory-b2c
ms.workload: identity
ms.topic: article
ms.date: 4/06/2017
ms.author: davidmu
ms.openlocfilehash: ee2d82f8c90b88a898428973a1febaa21034a14f
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/23/2018
---
# <a name="azure-active-directory-b2c-provide-sign-up-and-sign-in-to-consumers-with-twitter-accounts"></a>Azure Active Directory B2C: Zich kunnen registreren en aanmelden gebruikers bieden met Twitter-accounts

## <a name="create-a-twitter-application"></a>Een Twitter-toepassing maken
Als u wilt gebruiken als een id-provider in Azure Active Directory (Azure AD) B2C Twitter gebruikt, moet u een Twitter-toepassing maken en geeft deze met de juiste parameters. U moet een Twitter-ontwikkelaarsaccount om dit te doen. Als u niet hebt, kunt u krijgen op het [ https://dev.twitter.com/ ](https://dev.twitter.com/).

1. Ga naar de [Twitter-website voor ontwikkelaars van](https://dev.twitter.com/) en meld u aan met uw referenties.
2. Klik op **mijn apps** onder **hulpprogramma's en ondersteuning** en klik vervolgens op **nieuwe App maken**. 
3. Klik in het formulier geeft een waarde op voor de **naam**, **beschrijving**, en **Website**.
4. Voor de **retouraanroep URL**, voer `https://login.microsoftonline.com/te/{tenant}/oauth2/authresp`. Zorg ervoor dat u **{tenant}** met de naam van uw tenant (bijvoorbeeld contosob2c.onmicrosoft.com).
5. Schakel het selectievakje in om te accepteren de **Developer overeenkomst** en klik op **uw Twitter-toepassing maken**.
6. Zodra de app is gemaakt, klikt u op **sleutels en toegangstokens**.
7. Kopieer de waarde van **consumentsleutel** en **consumentgeheim**. U moet beide Twitter configureren als een id-provider in uw tenant.

## <a name="configure-twitter-as-an-identity-provider-in-your-tenant"></a>Twitter configureren als een id-provider in uw tenant
1. Volg deze stappen voor [gaat u naar de blade B2C-functies](active-directory-b2c-app-registration.md#navigate-to-b2c-settings) in de Azure portal.
2. Klik op de blade B2C-functies op **identiteitsproviders**.
3. Klik op **+Toevoegen** boven aan de blade.
4. Geef een beschrijvende **naam** voor de configuratie van de id-provider. Voer bijvoorbeeld 'Twitter'.
5. Klik op **identiteit providertype**, selecteer **Twitter**, en klik op **OK**.
6. Klik op **instellen van deze id-provider** en voer de Twitter **consumentsleutel** voor de **Client-id** en de Twitter **consumentgeheim** voor de **clientgeheim**.
7. Klik op **OK**, en klik vervolgens op **maken** naar uw Twitter-configuratie op te slaan.

## <a name="next-steps"></a>Volgende stappen

Creëer of bewerk een [ingebouwde beleid](active-directory-b2c-reference-policies.md) en Twitter toevoegen als een id-provider.