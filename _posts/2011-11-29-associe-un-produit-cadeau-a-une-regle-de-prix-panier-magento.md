---
layout: post
title:  Associé un produit cadeau a une règle de prix panier Magento"
author: ahmed
categories: [  tutorial,magento ]
image: assets/images/giftPromo1.jpg
featured: true
hidden: true
description: "Associé un produit cadeau a une règle de prix panier Magento"

---

**Nouveau un module magento** qui permet de lié un produit cadeau à une règle de prix panier,
Le module se nomme : **GiftPromo**.



**Le module permet :**
•	D’associé un produit cadeau a une règle de prix panier (règles de cadeaux) donc bénéficier de toutes les fonctionnalités des règles par exemple :
Associé un produit cadeau a un codes de remise, ajouter un produit cadeau lorsque le montant du panier est supérieur a une valeur bien déterminé, gérer les règles de cadeaux par priorité, groupes clients…..
•	Possibilité d’ajouter plusieurs règles de cadeaux
•	Possibilité d’ajouter plus d’un produit si règles est atteinte

Le module est disponible sur **magento Connect** : [GiftPromo](http://www.magentocommerce.com/magento-connect/amahi/extension/6559/s3ibusiness_giftpromo)

 

 

Donc Aujourd’hui on va voir commet utiliser **GiftPromo**, après l’installation il faut d’abord activer le module dans :
System -> configuration -> Ventes -> GiftPromo -> Paramétrage->Activé-> Oui

![règles de cadeaux]({{ site.baseurl }}/assets/images/configuration2.jpg)

Puis après aller dans :
Promotions -> Règles de prix panier-> Gestion des cadeaux.
Avant vous devez d’abord crée le produit (Catalogue/Gérer les produits) et n’oubliez pas de mettre un prix égale a zéro, une quantité positive du stock ou de choisir “Non” pour “Gérer les stocks” et noté l’Id du produit pour l’utiliser dans la création du cadeau.

![manage free gifts]({{ site.baseurl }}/assets/images/manage-gifts.jpg)
Puis ajouter des cadeaux:
![manage free gifts]({{ site.baseurl }}/assets/images/addgift.jpg)

Une fois les produits cadeaux ajouté, dans les “**Règles de prix panier**” vous pouvez crée des règles et associé un “produit cadeau” a cette règle :
![manage free gifts]({{ site.baseurl }}/assets/images/shiping-rule.jpg)

Dans l’onglet “Action”, -> “Mettre à jour les prix en utilisant l’information suivante” vous allez alors retrouver les produit cadeaux que vous avez crée dans la liste “Appliquer”, choisissez l’un d’eux pour l’associé a votre règle.
Note : ne pas changer la valeur de “Remise” et laissez-la à 0.
Et enfin vous pouvez tester l’ajout automatique de votre “produit cadeau” dans le panier dés que la règle est atteinte.
![cart free gift]({{ site.baseurl }}/assets/images/cart-gift.jpg)

