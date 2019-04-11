---
layout: post
title:  "Création d’un client a l’aide du code sous Magento"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/customer.jpg
---
Voyons comment créer un client.
Dans le helper par exemple j’ajoute la méthode :



```php
public function CreateCustomer($customerInformations) {
        $customer = Mage::getModel('customer/customer');
        $customer->setWebsiteId(Mage::app()->getWebsite()->getId());
        $customer->loadByEmail($customerInformations['email']);
        if (!$customer->getId()) {
            /* $customer->setEmail($customerInformations['email']);
              $customer->setFirstname($customerInformations['firstname']);
              $customer->setLastname($customerInformations['lastname']);
              $customer->setPassword($customerInformations['password']); */
            $customer->setData($customerInformations);
        }
 
        try {
            $customer->save();
        } catch (Exception $ex) {
            Zend_Debug::dump($ex->getMessage());
        }
        return $customer;
    }
```

Donc une petite explication de la méthode :
Premièrement j’initialise mon objet Model « $customer »

```php
$customer = Mage::getModel('customer/customer');
```

Puis je lui affect une boutique dans mon cas la boutique par défaut :

```php
$customer->setWebsiteId(Mage::app()->getWebsite()->getId());
```

Je vérifie si le client existe déjà dans la base (à l’aide d’adresse émail)

```php
 $customer->loadByEmail($customerInformations['email']);
        if (!$customer->getId()) {
```

Pour renseigner les informations client je peux utiliser soit:

```php
$customer->setEmail($customerInformations['email']);
$customer->setFirstname($customerInformations['firstname']);
$customer->setLastname($customerInformations['lastname']);
$customer->setPassword($customerInformations['password']);
```

Ou

```php
$customerInformations = array(
            'email' => '1hmedmahi@gmail.com',
            'firstname' => 'Ahmed',
            'lastname' => 'Mahi',
            'password' => 'magento');
Mage::helper('inforbycode')->CreateCustomer($customerInformations);
```

On fait on pas encore terminé car on a juste crée un client avec des informations basique il ne reste ses adresses.
A suivre…. ![:)]({{ site.baseurl }}/assets/images/icon_smile.gif)

