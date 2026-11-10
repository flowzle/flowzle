# CONTEXTE — Flowzle (site propre)

Journal du projet, entrée la plus récente en haut. Identifiants et réglages non secrets :
`projet.env`. Dépôt du site : `../website`.

---

## 2026-11-10 — Branchements Google : GSC, GA4, GTM, bandeau de consentement

**Fait**

- **Search Console** : propriété de domaine `sc-domain:flowzle.com` vérifiée par un TXT à l'apex,
  posé par Terraform (`website/infrastructure/dns-mail.tf`, ressource `apex_txt`, valeur ajoutée à
  la liste existante pour ne pas casser le SPF). `terraform apply` passé en production. Sitemap
  `sitemap-index.xml` soumis — erreur « Impossible de récupérer le sitemap » au moment de la
  soumission, attribuée à une latence côté Google sur une propriété fraîche (WAF, redirections et
  user-agent Googlebot écartés par test). **À recontrôler.**
- **GA4** : propriété et flux web créés. Conservation portée à 14 mois, Google Signals laissé
  désactivé, « Interactions avec les formulaires » décoché dans la mesure améliorée.
- **GTM** : conteneur créé, plan de marquage importé et **publié** (4 balises, 3 déclencheurs,
  2 variables). `cookie_expires = 34128000` posé pour tenir la durée de 13 mois annoncée dans la
  politique de confidentialité.
- **Bandeau de consentement** : Consent Mode v2 en mode basique. Les trois paramètres publicitaires
  sont refusés en toutes circonstances ; GTM n'est chargé qu'après acceptation. Vérifié sur le build
  servi : **zéro** balise Google chargée d'office.
- **Marquage du formulaire** : `contact_envoye`, `contact_erreur` (+ `champs_en_erreur`) et
  `clic_email` poussés dans le `dataLayer`. Seuls des noms de champs circulent, jamais les saisies.
- **Pages légales** : politique de confidentialité complétée (GA4, finalité, base légale, cookies,
  durées, Google Ireland, transferts hors UE) et durées de conservation arrêtées — trois ans après
  le dernier échange pour une demande de contact. `donneesPersonnellesEtCookies` passé à `true`.
- **Outillage** : `gh` rendu utilisable (il était installé mais hors du PATH de session).

**Deux pièges rencontrés, documentés dans le socle**

1. Le chargement maison de GTM omettait `dataLayer.push({ 'gtm.start', event: 'gtm.js' })`. Le
   conteneur se chargeait, les événements personnalisés fonctionnaient, mais **aucune page vue** :
   toute balise sur « Toutes les pages » restait muette. Corrigé (`v1.3.1`).
2. La « balise Google » auto-créée avec la propriété GA4 (`GTM-K9X5GD6T`) s'affiche comme un
   conteneur sans en être un. Ne pas la confondre avec `GTM-WHKQRKRJ`.

**Versions déployées** : `v1.3.0` (bandeau + câblage), `v1.3.1` (correctif page vue),
`v1.3.2` (politique de confidentialité), `v1.3.3` (durées de conservation).

**Fausse alerte** : une notification du formulaire non reçue à `contact@flowzle.com`. Rebond
**transitoire** unique (greylisting OVH probable) ; les envois suivants sont passés. DKIM valide,
DNS intact, compte SES sain.

**Reste à faire**

- [ ] Étoiler `contact_envoye` et `clic_email` comme événements clés dans GA4 (à partir du
      11/11/2026, délai de remontée), et décocher `purchase`, `qualify_lead`, `close_convert_lead`.
- [ ] Recontrôler la récupération du sitemap dans la Search Console.
- [ ] **Visibilité des rebonds SES** : aucun « configuration set », donc aucun motif de rebond
      enregistré. Une demande peut se perdre sans que personne ne le sache.
- [ ] **Redirection `www` → apex** : demande du 10/11, nécessite une modification du module
      `fondations`, une `v0.0.5` et un bump du `ref=` dans `website/infrastructure/main.tf`.
      Note : `formulaire.origines` autorise encore `https://www.flowzle.com`, à revoir ensuite.
- [ ] **Incohérence du README** : `AWS_PROFILE=aoni-studio-dev-admin` alors que le seul profil
      configuré est `flowzle` (SSO, compte 369488682724).
- [ ] **Refaire tous les branchements sous le compte Google pro Flowzle** — tout est aujourd'hui
      sur le compte personnel de James. Tant que ce n'est pas fait, ne pas monter le projet Google
      Cloud ni le client OAuth des skills (`gsc-manager`, `ga4-manager`) : ce serait à refaire.
