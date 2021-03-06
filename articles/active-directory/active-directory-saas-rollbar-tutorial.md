---
title: 'Zelfstudie: Azure Active Directory-integratie met Rollbar | Microsoft Docs'
description: Informatie over het configureren van eenmalige aanmelding tussen Azure Active Directory en Rollbar.
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 57537e54-9388-4272-a610-805ce45a451f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 1/04/2017
ms.author: jeedes
ms.openlocfilehash: 43dc50d0a5381ace8bcfeb3cae39e249ba743876
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 02/28/2018
---
# <a name="tutorial-azure-active-directory-integration-with-rollbar"></a>Zelfstudie: Azure Active Directory-integratie met Rollbar

In deze zelfstudie leert u hoe Rollbar integreren met Azure Active Directory (Azure AD).

Rollbar integreren met Azure AD biedt de volgende voordelen:

- U kunt beheren in Azure AD die toegang tot Rollbar heeft.
- U kunt uw gebruikers automatisch ophalen aangemeld bij Rollbar (Single Sign-On) met hun Azure AD-accounts kunt inschakelen.
- U kunt uw accounts op één centrale locatie - en de Azure-portal beheren.

Als u weten van meer informatie over de integratie van de SaaS-app met Azure AD wilt, Zie [wat is er toegang tot toepassingen en eenmalige aanmelding bij Azure Active Directory](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Vereisten

Voor het configureren van Azure AD-integratie met Rollbar, moet u de volgende items:

- Een Azure AD-abonnement
- Een Rollbar eenmalige aanmelding ingeschakeld abonnement

> [!NOTE]
> Test de stappen in deze zelfstudie, raden we niet met behulp van een productieomgeving.

Test de stappen in deze zelfstudie, moet u deze aanbevelingen volgen:

- Gebruik niet uw productieomgeving, tenzij het noodzakelijk is.
- Als u geen een proefabonnement Azure AD-omgeving hebt, kunt u [ophalen van een proefversie van één maand](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Scenariobeschrijving
In deze zelfstudie test u Azure AD eenmalige aanmelding in een testomgeving. Het scenario in deze zelfstudie bestaat uit twee belangrijkste bouwstenen:

1. Rollbar uit de galerie toevoegen
2. Configureren en testen van Azure AD eenmalige aanmelding

## <a name="adding-rollbar-from-the-gallery"></a>Rollbar uit de galerie toevoegen
Voor het configureren van de integratie van Rollbar in Azure AD, moet u Rollbar uit de galerie toevoegen aan de lijst met beheerde SaaS-apps.

**Als u wilt toevoegen Rollbar uit de galerie, moet u de volgende stappen uitvoeren:**

1. In de  **[Azure-portal](https://portal.azure.com)**, klik in het linkernavigatievenster op **Azure Active Directory** pictogram. 

    ![De Azure Active Directory-knop][1]

2. Navigeer naar **bedrijfstoepassingen**. Ga vervolgens naar **alle toepassingen**.

    ![De blade Enterprise-toepassingen][2]
    
3. Om de nieuwe toepassing toevoegen, klikt u op **nieuwe toepassing** knop boven aan het dialoogvenster.

    ![De knop Nieuw toepassing][3]

4. Typ in het zoekvak **Rollbar**, selecteer **Rollbar** van resultaat deelvenster klik vervolgens op **toevoegen** om toe te voegen van de toepassing.

    ![Rollbar in de lijst met resultaten](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configureren en testen eenmalige aanmelding Azure AD

In deze sectie configureert en test eenmalige aanmelding Azure AD met Rollbar op basis van een testgebruiker 'Britta Simon' genoemd.

Voor eenmalige aanmelding werkt, moet Azure AD weten wat de gebruiker equivalent in Rollbar is voor een gebruiker in Azure AD. Met andere woorden, moet een koppeling relatie tussen een Azure AD-gebruiker en de betreffende gebruiker in Rollbar tot stand worden gebracht.

Wijs in Rollbar, de waarde van de **gebruikersnaam** in Azure AD als de waarde van de **gebruikersnaam** de relatie van de koppeling tot stand brengen.

Om te configureren en testen van Azure AD eenmalige aanmelding met Rollbar, moet u de volgende bouwstenen voltooien:

1. **[Azure AD eenmalige aanmelding configureren](#configure-azure-ad-single-sign-on)**  : als u wilt dat uw gebruikers kunnen deze functie gebruiken.
2. **[Maken van een Azure AD-testgebruiker](#create-an-azure-ad-test-user)**  - voor het testen van Azure AD eenmalige aanmelding met Britta Simon.
3. **[Maak een testgebruiker Rollbar](#create-a-rollbar-test-user)**  - Rollbar die is gekoppeld aan de Azure AD-weergave van de gebruiker van een exemplaar van Britta Simon bevatten.
4. **[Toewijzen van de Azure AD-testgebruiker](#assign-the-azure-ad-test-user)**  - Britta Simon gebruik van Azure AD eenmalige aanmelding inschakelen.
5. **[Test eenmalige aanmelding](#test-single-sign-on)**  : om te controleren of de configuratie werkt.

### <a name="configure-azure-ad-single-sign-on"></a>Eenmalige aanmelding Azure AD configureren

In dit gedeelte Azure AD eenmalige aanmelding inschakelen in de Azure portal en eenmalige aanmelding in uw toepassing Rollbar configureren.

**Voor het configureren van Azure AD eenmalige aanmelding met Rollbar, moet u de volgende stappen uitvoeren:**

1. In de Azure-portal op de **Rollbar** toepassing Integratiepagina, klikt u op **eenmalige aanmelding**.

    ![Koppeling voor eenmalige aanmelding configureren][4]

2. Op de **eenmalige aanmelding** dialoogvenster Selecteer **modus** als **op basis van SAML aanmelding** voor eenmalige aanmelding inschakelen.
 
    ![Dialoogvenster voor eenmalige aanmelding](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_samlbase.png)

3. Op de **Rollbar domein en de URL's** sectie, voert u de volgende stappen uit als u wilt configureren, de toepassing in **IDP** modus gestart:

    ![URL's en Rollbar domein eenmalige aanmelding informatie](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_url.png)

    a. In de **id** textbox, typ de URL: `https://saml.rollbar.com`

    b. In de **antwoord-URL** textbox, typ een URL met het volgende patroon volgen: `https://rollbar.com/<accountname>/saml/sso/azure/`

4. Controleer **weergeven geavanceerde instellingen voor URL** en voer de volgende stap als u wilt configureren van de toepassing in **SP** modus gestart:

    ![URL's en Rollbar domein eenmalige aanmelding informatie](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_url1.png)

    In de **aanmeldings-URL** textbox, typ een URL met het volgende patroon volgen: `https://rollbar.com/<accountname>/saml/login/azure/`
     
    > [!NOTE] 
    > Deze waarden zijn niet echt. Deze waarden bijwerken met de werkelijke antwoord-URL en de aanmeldings-URL. Neem contact op met [Rollbar Client ondersteuningsteam](mailto:support@rollbar.com) ophalen van deze waarden. 

5. Op de **SAML-certificaat voor ondertekening van** sectie, klikt u op **Metadata XML** en sla het bestand met metagegevens op uw computer.

    ![De downloadkoppeling certificaat](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_certificate.png) 

6. Klik op **opslaan** knop.

    ![Knop Single Sign-On opslaan configureren](./media/active-directory-saas-rollbar-tutorial/tutorial_general_400.png)
    
7. In een ander browservenster, meld u aan bij uw bedrijf Rollbar site als beheerder.

8. Klik op de **profielinstellingen** in de rechterbovenhoek en klik op **accountnaam instellingen**.
    
    ![Configuratie](./media/active-directory-saas-rollbar-tutorial/general.png)

9. Klik op **identiteitsprovider** onder beveiliging.

    ![Configuratie](./media/active-directory-saas-rollbar-tutorial/configure1.png)

10. In de **SAML-identiteitsprovider** sectie, voert u de volgende stappen uit:
    
    ![Configuratie](./media/active-directory-saas-rollbar-tutorial/configure2.png)

    a. Selecteer **AZURE** van de **SAML-identiteitsprovider** vervolgkeuzelijst.

    b. Open uw metagegevensbestand in Kladblok, Kopieer de inhoud ervan naar het Klembord en plakt u deze naar de **SAML metagegevens** textbox.

    c. Klik op **Opslaan**.

11. Na het opslaan te klikken op knop, het scherm worden als volgt:
    
    ![Configuratie](./media/active-directory-saas-rollbar-tutorial/configure3.png)
    > [!NOTE] 
    > Als u wilt de volgende stap hebt voltooid, moet u eerst toevoegen zelf als een gebruiker in de Rollbar-app in Azure.
    a. Als u wilt dat alle gebruikers moeten verifiëren via Azure en klik vervolgens op **aanmelden via uw identiteitsprovider** opnieuw via Azure worden geverifieerd.  

    b.  Als u wordt teruggeleid naar het scherm, selecteert u de **is aanmelding via SAML-identiteitsprovider vereist** selectievakje.

    b. Klik op **Opslaan**.

> [!TIP]
> U kunt nu een beknopte versie van deze instructies binnen lezen de [Azure-portal](https://portal.azure.com), terwijl u de app instelt!  Na het toevoegen van deze app uit de **Active Directory > bedrijfstoepassingen** sectie, klikt u op de **Single Sign-On** tabblad en toegang tot de ingesloten documentatie via de **configuratie** sectie onderaan. U kunt meer lezen over de ingesloten documentatie-functie: [embedded-documentatie voor Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>Een Azure AD-testgebruiker maken

Het doel van deze sectie is het een testgebruiker maken in de Azure portal Britta Simon aangeroepen.

   ![Een Azure AD-testgebruiker maken][100]

**Als u wilt een testgebruiker maken in Azure AD, moet u de volgende stappen uitvoeren:**

1. Klik in de Azure-portal in het linkerdeelvenster op het **Azure Active Directory** knop.

    ![De Azure Active Directory-knop](./media/active-directory-saas-rollbar-tutorial/create_aaduser_01.png)

2. Als u wilt weergeven in de lijst met gebruikers, gaat u naar **gebruikers en groepen**, en klik vervolgens op **alle gebruikers**.

    !['Gebruikers en groepen' en 'Alle gebruikers' koppelingen](./media/active-directory-saas-rollbar-tutorial/create_aaduser_02.png)

3. Openen van de **gebruiker** in het dialoogvenster klikt u op **toevoegen** boven aan de **alle gebruikers** in het dialoogvenster.

    ![De knop toevoegen](./media/active-directory-saas-rollbar-tutorial/create_aaduser_03.png)

4. In de **gebruiker** dialoogvenster vak, voert u de volgende stappen uit:

    ![Het dialoogvenster gebruiker](./media/active-directory-saas-rollbar-tutorial/create_aaduser_04.png)

    a. In de **naam** in het vak **BrittaSimon**.

    b. In de **gebruikersnaam** typt u het e-mailadres van gebruiker Britta Simon.

    c. Selecteer de **wachtwoord weergeven** selectievakje, en noteer de waarde die wordt weergegeven in de **wachtwoord** vak.

    d. Klik op **Create**.
 
### <a name="create-a-rollbar-test-user"></a>Een testgebruiker Rollbar maken

Om Azure AD-gebruikers zich aanmelden bij Rollbar, moeten ze worden ingericht in Rollbar. In het geval van Rollbar is inrichting een handmatige taak.

**Voor het inrichten van een gebruikersaccount, moet u de volgende stappen uitvoeren:**

1. Meld u aan bij uw bedrijf Rollbar site als beheerder.

2. Klik op de **profielinstellingen** in de rechterbovenhoek en klik op **accountnaam instellingen**.

    ![Gebruiker](./media/active-directory-saas-rollbar-tutorial/general.png)

3. Klik op **gebruikers**.
    
    ![Werknemer toevoegen](./media/active-directory-saas-rollbar-tutorial/user1.png)

4. Klik op **uitnodigen teamleden**.

    ![Personen uitnodigen](./media/active-directory-saas-rollbar-tutorial/user2.png)

5. Voer de naam van gebruiker, zoals in het tekstvak  **brittasimon@contoso.com**  en klik op **toevoegen/uitnodiging**.

    ![Personen uitnodigen](./media/active-directory-saas-rollbar-tutorial/user3.png)

6. Gebruiker ontvangt een uitnodiging voor en na accepteren hij gemaakt in het systeem.

### <a name="assign-the-azure-ad-test-user"></a>De Azure AD-testgebruiker toewijzen

In deze sectie schakelt u Britta Simon gebruikt Azure eenmalige aanmelding toegang verlenen aan Rollbar.

![Toewijzen van de gebruikersrol][200] 

**Britta Simon om aan te wijzen Rollbar, moet u de volgende stappen uitvoeren:**

1. Open de weergave toepassingen in de Azure-portal en gaat u naar de directoryweergave en gaat u naar **bedrijfstoepassingen** klikt u vervolgens op **alle toepassingen**.

    ![Gebruiker toewijzen][201] 

2. Selecteer in de lijst met toepassingen **Rollbar**.

    ![De koppeling Rollbar in de lijst met toepassingen](./media/active-directory-saas-rollbar-tutorial/tutorial_rollbar_app.png)  

3. Klik in het menu aan de linkerkant op **gebruikers en groepen**.

    ![De koppeling 'Gebruikers en groepen'][202]

4. Klik op **toevoegen** knop. Selecteer vervolgens **gebruikers en groepen** op **toevoegen toewijzing** dialoogvenster.

    ![Het deelvenster toewijzing toevoegen][203]

5. Op **gebruikers en groepen** dialoogvenster Selecteer **Britta Simon** in de lijst gebruikers.

6. Klik op **Selecteer** knop op **gebruikers en groepen** dialoogvenster.

7. Klik op **toewijzen** knop op **toevoegen toewijzing** dialoogvenster.
    
### <a name="test-single-sign-on"></a>Test eenmalige aanmelding

In deze sectie kunt u uw Azure AD eenmalige aanmelding configuratie met behulp van het toegangsvenster testen.

Als u op de tegel Rollbar in het deelvenster toegang, u moet ophalen automatisch aangemeld bij uw toepassing Rollbar.
Zie voor meer informatie over het toegangsvenster [Inleiding tot het toegangsvenster](active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Aanvullende resources

* [Lijst met zelfstudies over het integreren van SaaS-Apps met Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Wat is de toegang tot toepassingen en eenmalige aanmelding bij Azure Active Directory?](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_04.png

[100]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_202.png
[203]: ./media/active-directory-saas-rollbar-tutorial/tutorial_general_203.png

