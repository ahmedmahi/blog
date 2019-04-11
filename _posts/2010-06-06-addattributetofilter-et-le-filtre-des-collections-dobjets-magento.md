---
layout: post
title:  "addAttributeToFilter() et le filtre des collections d’objets Magento"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/magento.jpg
---
Dans une série de petit articles sur la manipulation des collections d’objets Magento  aujourd’hui on va parler sur la fonction **addAttributeToFilter()** cette fonction sert à filtrer les  collections d’objets (Entités) suivant leur attributs (on parle ici des objets  stockés suivant  le modèle EAV :**Entité – Attribut – Valeur**) :

Par exemple on cherche les produits activés dans le BackOffice donc dans ce cas on utilise :

```php
$_productCollection = $model_product->getCollection()
addAttributeToFilter('status', 1)
```

Maintenant les différents type de filtre pour fonction addAttributeToFilter() :

```php
array("from"=>$fromValue, "to"=>$toValue)
array("like"=>$likeValue)
array("neq"=>$notEqualValue)
array("in"=>array($inValues))
array("nin"=>array($notInValues))
```

Un exemple : on filtre sur les produits dont le nom commence par la lettre a:

```php
addAttributeToFilter('name',array('like'=>'a%'))
```

