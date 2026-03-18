# TODO.md — Compare_Object Audit

**Audit date:** 2026-03-18
**Total findings:** 46 (3 Critical · 10 Major · 25 Minor · 8 Suggestion)

---

## 🔴 Critical

| # | Cat | Fichier | Ligne | Finding | Fix |
|---|-----|---------|-------|---------|-----|
| CO-001 | 4-Sécurité | `_c02` | — | **Aucun AUTHORITY-CHECK** dans tout le programme. Le report se connecte via RFC à des systèmes distants, lit des tables DD (TADIR, TDEVC, TFDIR, DD03VT) et des tables customizing, sans aucun contrôle d'accès. | Ajouter `AUTHORITY-CHECK OBJECT 'S_RFC' ID 'ACTVT' FIELD '16'` avant chaque appel RFC ; ajouter `S_TABU_DIS ACTVT=03` pour la lecture des tables |
| CO-002 | 2-Erreur | `_c02` | 1924–2094 | **18 blocs `CATCH cx_salv_not_found` vides** (colonnes ALV non trouvées, silencieusement ignorées). L'ALV s'affiche sans les colonnes configurées et l'utilisateur n'en est pas informé. | Ajouter `##NO_HANDLER` explicite si comportement voulu, sinon logger le nom de colonne manquante avec `MESSAGE` |
| CO-003 | 2-Erreur | `_c02` | 2111 | **`CATCH cx_root. "#EC NOHANDLER`** — avale silencieusement toutes les exceptions d'un bloc de traitement principal. Toute erreur inattendue est perdue. | Remplacer par types d'exceptions spécifiques ; logger avant de supprimer |

---

## 🟠 Major

| # | Cat | Fichier | Ligne | Finding | Fix |
|---|-----|---------|-------|---------|-----|
| CO-004 | 4-Sécurité | `_c02` | 3884, 3919 | **Appel dynamique `CALL METHOD me->(lv_method)`** sans validation de `lv_method`. Si la valeur dérive d'une entrée utilisateur ou d'une table, risque d'exécution de méthode non prévue. | Valider `lv_method` contre une liste blanche (`CASE`/`SWITCH`) avant l'appel dynamique |
| CO-005 | 4-Sécurité | `_c02` | 188 | **Destination RFC non validée** avant les appels. L'utilisateur peut saisir n'importe quelle destination RFC (param. `p_rfc_o` / `p_rfc_t`). | Vérifier la destination RFC contre une liste de systèmes autorisés avant toute connexion |
| CO-006 | 8-Incohérence | `_c02` | 1545 | **Double affectation avec mismatch de types** : `lv_percentage = lv_integer = |{ sy-tabix ZERO = NO }|.` — `lv_integer` est de type `INT4` mais reçoit un template STRING. Risque d'erreur runtime si la conversion implicite échoue. | Séparer : `lv_integer = sy-tabix. lv_percentage = |{ lv_integer }|.` |
| CO-007 | 2-Erreur | `_c02` | 2301–2486 | **14 blocs `CATCH cx_root. "#EC NO_HANDLER`** avec commentaires "Tant pis" — aucun log, aucune récupération. Échecs silencieux sur configuration ALV colonnes. | Ajouter au minimum `DATA(lv_err_text) = cx->get_text( ).` + `MESSAGE ... TYPE 'W'` |
| CO-008 | 2-Erreur | `_c02` | 2494, 2611, 2642 | **3 blocs `CATCH cx_salv_msg`** "On verra" / "On verra" — création ALV échoue silencieusement. L'utilisateur voit un écran vide sans raison. | Logger l'erreur ALV ; afficher un message à l'utilisateur avant `RETURN` |
| CO-009 | 2-Erreur | `_c02` | 3537 | **`CATCH cx_root.`** sur création dynamique de champ — erreur ignorée sans message. | Logger le contexte (nom du champ) avant suppression |
| CO-010 | 2-Erreur | `_c02` | 3601, 3609, 3617 | **CATCH `cx_sy_struct_creation` / `cx_sy_table_creation` / `cx_root`** — flag d'erreur positionné mais aucun message affiché à l'utilisateur. | Ajouter `MESSAGE` informatif sur l'objet en cours de création |
| CO-011 | 2-Erreur | `_c02` | 3889, 3924 | **`CATCH cx_root.`** sur appels de méthodes dynamiques — échec ignoré avec simple `RETURN`. | Logger le nom de méthode (`lv_method`) et le texte d'exception avant `RETURN` |
| CO-012 | 3-Performance | `_c02` | 1537–1605 | **Appels RFC répétés dans une LOOP** — `_rfc_read_table` / `_rfc_table_lines_count` appelés pour chaque objet comparé. Risque de dégradation si >100 objets. | Vérifier le nombre d'itérations max ; documenter la limite ; envisager traitement par lot (`PACKAGE SIZE`) |
| CO-013 | 6-Hardcode | `_c02` | 188 | **`|{ sy-sysid }CLNT{ sy-mandt }|`** — le système d'origine est fixé au système courant, non paramétrable. Impossible d'utiliser le report depuis un 3e système. | Permettre la surcharge via un paramètre optionnel sur l'écran de sélection |

---

## 🟡 Minor

| # | Cat | Fichier | Ligne | Finding | Fix |
|---|-----|---------|-------|---------|-----|
| CO-014 | 1-Naming | `_c02` | multiple | **41 méthodes de la classe locale en minuscules** (ex: `initialization`, `start_of_selection`, `_check_input`…). Convention ABAP : UPPERCASE_UNDERSCORE. | Renommer toutes les méthodes en UPPERCASE dans la définition (`_c01`) et l'implémentation (`_c02`) |
| CO-015 | 1-Naming | `_top` | 58–59 | **Typo variables globales** : `gv_sytem_target` / `gv_sytem_origin` (manque un 's') → `gv_system_target` / `gv_system_origin`. | Corriger dans `_top` et toutes les références dans `_c02` |
| CO-016 | 1-Naming | `_top` | 30–31 | **`DATA gc_rowcount`** déclaré dans une section "CONSTANTES" : doit être `CONSTANTS : gc_rowcount TYPE i VALUE 200.` | Remplacer `DATA` par `CONSTANTS` |
| CO-017 | 1-Naming | `_c01`, main | 36 / 57 | **Typo méthode** `selection_screen_ouput` (→ `selection_screen_output`) — propagé dans l'appel du programme principal. | Corriger dans la définition (`_c01`) et dans l'appel (`main prog. ligne 57`) |
| CO-018 | 1-Naming | `_c01` | 283 | **Typo type** `ts_compare_wokbench_de_tab` → `ts_compare_workbench_de_tab` | Corriger |
| CO-019 | 1-Naming | tous | header | **Tous les includes ont un commentaire header incorrect** : `Z_A_BC_POST_REFRESH_CHECK_*` au lieu de `Z_A_BC_COMPARE_OBJECT_*` (ancien nom du programme). | Mettre à jour les commentaires en tête de chaque include |
| CO-020 | 1-Naming | main | 2–4 | **En-tête programme** indique `Z_A_BC_POST_REFRESH_CHECK` au lieu de `Z_A_BC_COMPARE_OBJECT`. | Corriger |
| CO-021 | 5-Dead code | main | 34 | **INCLUDE commenté** `z_a_bc_compare_object_cta` (include de parallélisation). Aucun commentaire explicatif. | Documenter la raison de la désactivation ou supprimer |
| CO-022 | 5-Dead code | `_sel` | 44–45 | **`SELECT-OPTIONS s_trkorr`** (filtre par ordre de transport) commenté sans explication. | Documenter ou supprimer |
| CO-023 | 5-Dead code | `_c01` | 103 | **Constante `mc_archive_type`** commentée — probablement liée au bloc archive désactivé. | Supprimer ou documenter |
| CO-024 | 5-Dead code | `_c02` | 3702–3717 | **22 lignes commentées** : bloc d'archivage `zcl_archive=>archive_add(...)` + `CATCH zcx_archive` non implémenté. | Supprimer ou réactiver avec gestion d'erreur complète |
| CO-025 | 5-Dead code | `_c02` | 1259, 1440–1445 | **Paramètres FM commentés** (`DELIMITER`, plusieurs paramètres `range_2_where`) sans explication. | Documenter la raison de l'omission |
| CO-026 | 2-Erreur | `_c02` | 599, 679 | **`CATCH cx_sy_itab_line_not_found`** sans log ni pragma — intention peu claire. | Ajouter `##NO_HANDLER` si intentionnel |
| CO-027 | 2-Erreur | `_c02` | 1706–1709 | **`CATCH cx_uuid_error`** avec fallback silencieux "Tant pis" — UUID generation échoue sans avertissement. | Logger l'avertissement avant le fallback |
| CO-028 | 2-Erreur | `_c02` | 2662 | **`CATCH cx_salv_not_found`** colonne non trouvée — ignorée sans log du nom de colonne. | Logger le nom de colonne avant suppression |
| CO-029 | 6-Hardcode | `_c02` | 1920–2100 | **Noms de colonnes ALV codés en dur** (`'ID_RESULT'`, `'UNAME'`, `'TIMESTAMP'`, etc.) dans ~20 appels `set_color_column`, `set_column_visible`… | Créer des constantes `GC_COL_*` dans `lcl_main` |
| CO-030 | 6-Hardcode | `_c02` | 951–963 | **Noms de champs codés en dur** (`'TABNAME'`, `'MAINFLAG'`, `'CONTFLAG'`) avec `##NOTEXT`. | Créer constantes nommées |
| CO-031 | 3-Performance | `_c02` | 3220 | **`SELECT FOR ALL ENTRIES` sur `dd03vt`** sans vérification d'index. La table peut être volumineuse si beaucoup de composants. | Vérifier l'existence d'index sur `(tabname, fieldname, ddlanguage)` |
| CO-032 | 3-Performance | `_c02` | 969 | **`CONDENSE lv_where_clause`** potentiellement exécuté en boucle — à vérifier. | Confirmer le contexte d'exécution |
| CO-033 | 8-Incohérence | `_c02` | 2111 vs 2301+ | **Pragma incohérent** : `#EC NOHANDLER` (ligne 2111) vs `#EC NO_HANDLER` (lignes 2301+) — deux orthographes différentes. | Standardiser sur `##NO_HANDLER` (syntaxe moderne) |
| CO-034 | 8-Incohérence | `_c02` | global | **Style de commentaires mixte** : `" x`, `"" --> x`, `"" Tant pis` — lecture difficile. | Standardiser le style |
| CO-035 | 1-Naming | `_c01` | 7 | **Typo commentaire** `DEFINIION` → `DEFINITION`. | Corriger |
| CO-036 | 1-Naming | `_c01` | 59 | **`PREFERRED PARAMETER`** sur `!iv_tabname` alors que `!iv_row` est déclaré en premier — cohérence à vérifier. | Vérifier l'utilisation effective ; retirer si inutile |
| CO-037 | 1-Naming | `_sel` | 83 | **Typo commentaire** `identifuqe` → `identique`. | Corriger |
| CO-038 | 4-Sécurité | `_c02` | — | **Données exportées sans log d'audit** — l'archive est désactivée (CO-024). Un accès non autorisé ne laisse aucune trace. | Réactiver l'archive ou prévoir un log applicatif |

---

## 🔵 Suggestion

| # | Cat | Fichier | Ligne | Finding | Fix |
|---|-----|---------|-------|---------|-----|
| CO-039 | 7-Modern | `_c02` | 36, 1879, 2249, 2281, 2594 | **5 × `CREATE OBJECT`** → `NEW #( )` / `NEW lcl_xxx( )` | `ro_main = NEW #( ).` etc. |
| CO-040 | 7-Modern | `_c02` | 503, 507 | **`ADD 1 TO` / `SUBTRACT 1 FROM`** → opérateurs `+=` / `-=` | `ls_selected_rows += 1.` |
| CO-041 | 7-Modern | `_c02` | 3036 | **`MOVE-CORRESPONDING`** → `CORRESPONDING #( )` | `<lfs_s_display> = CORRESPONDING #( <lfs_s_content> ).` |
| CO-042 | 7-Modern | `_c02` | 1546–1550 | **Plusieurs `REPLACE '&N' WITH ... INTO`** → un seul template string | Consolidation en template `| ... { lv_xxx } ... |` |
| CO-043 | 7-Modern | `_c02` | multiple | **Concaténations avec `&&`** → templates `| ... |` plus lisibles | Standardiser sur string templates |
| CO-044 | 7-Modern | `_c01` | 72–101 | **Structure de constantes `BEGIN OF / END OF`** → déclarations inline ABAP 7.40 | Moderniser si lisibilité améliorée |
| CO-045 | 8-Incohérence | `_c01` | 246 | **`TYPES ... INCLUDE TYPE`** dans structure imbriquée — peu lisible | Aplatir la définition de type |
| CO-046 | 8-Incohérence | `_c01` | 568–574 | **Section `PRIVATE` désorganisée** — DATA après bloc TYPES, commentaires redondants | Restructurer : TYPES → CONSTANTS → DATA → METHODS |
