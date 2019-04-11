---
layout: post
title:  "Ajouter des champs spécifiques dans l’inscription d’un nouveau client"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/champs-specifiques-commande.jpg
---
Des fois on a besoin d’avoir quelques informations supplémentaires sur nos clients par exemple : n° Siret, date de naissance… aujourd’hui donc on va essayer de voir comment ajouter des champs spécifiques dans le formulaire d’inscription des clients.

Voici la structure de notre petit module :

![structure du module magento: attribut cleint]({{ site.baseurl }}/assets/images/structure.png)

Sans oublier le fichier Mahigento_Newattcustomer.xml pour l’activation de notre nouveau module.
Premièrement on doit ajouter un nouvel attribut client :
Dans le fichier mysql4-install-0.1.0.php vous allez mettre :

```php
<?php
$installer=new  Mage_Customer_Model_Entity_Setup ('core_setup');
$installer->startSetup();

$installer->addAttribute('customer', 'siret', array(
    'label'        => 'Siret',
    'visible'      => true,
    'required'     => false,
   ));

$installer->endSetup(); 
```

Cela lors de l’installation du module va crée automatiquement l’attribut client “siret”.
![attribut client backoffice  Magento]({{ site.baseurl }}/assets/images/attribut-client-backoffice.png)

c’est fini avec le backoffice passons maintenant au front ![:)]({{ site.baseurl }}/assets/images/icon_smile.gif)
Maintenant le frontOffice on ajoute notre champ dans le fichier templete il s’agit de : customer/form/register.phtml
On ajoute par exemple :

```html
<li>                            
<label for="siret" class="required"><em>*</em>
    <?php echo $this->__('Siret') ?>
 </label>
<div class="input-box">                            
<input type="text" name="siret" id="siret"  
      title="<?php echo $this->__('Siret') ?>" class="input-text  required-entry" />
</div>                            
</li>
```

Puis il nous reste la partie traitement de la valeur du champ siret lors d’enregistrement du client, pour ça on a deux choix soit surcharger la méthode createPostAction() de la class Mage_Customer_AccountController ( ce que je préfère pas ![:)]({{ site.baseurl }}/assets/images/icon_smile.gif) ) soit en utilisant un observer, on va utiliser la deuxième méthode.
Dans notre fichier de configuration :

```xml
<?xml version="1.0"?>
<config>
    <modules>
        <Mahigento_Newattcustomer>
            <version>0.1.0</version>
        </Mahigento_Newattcustomer>
    </modules>
    <global>
        <events>
            <controller_action_postdispatch>
                <observers>
                    <mahigento_newattcustomer_save>
                        <type>singleton</type>
                        <class>newattcustomer/observer</class>
                        <method>saveSiret</method>
                    </mahigento_newattcustomer_save>
                </observers>
            </controller_action_postdispatch>
        </events>
        <models>
            <newattcustomer>
                <class>Mahigento_Newattcustomer_Model</class>
            </newattcustomer>
        </models>
        <resources>
            <newattcustomer_setup>
                <setup>
                    <module>Mahigento_Newattcustomer</module>
                </setup>
                <connection>
                    <use>core_setup</use>
                </connection>
            </newattcustomer_setup>
        </resources>
    </global>
</config>
```

et dans Observer.php:

```php
<?php

class Mahigento_Newattcustomer_Model_Observer {

    public function saveSiret($observer) {
        $Controller = $observer->getControllerAction();
        //si notre Controller correspond bien au controlller qui traite l'enregistrement
        if ($Controller instanceof Mage_Customer_AccountController) {
            $actionName = $Controller->getFullActionName();
            // l'action qui traite l'enregistrement d'un nouveu cleint
            if ($actionName == 'customer_account_createpost') {
                // récuperer les informations envoyé par le formulaire
                $data = $Controller->getRequest()->getPost();
                $customerId=Mage::getSingleton('customer/session')->getId();
                $customer = Mage::getModel('customer/customer')->load($customerId);
                // affecter le siret au client en cours
                $customer->setSiret($data['siret']);
                $customer->save();
            }
        }
    }

}
```

Et voila maintenant on a le champ Siret ajouter dans le formulaire d’enregistrement et dans le backoffice Magento comme information supplémentaire du client.

Testé sur 1.4.1.0 et 1.4.1.1
[Télécharger les fichiers]({{ site.baseurl }}/assets/files/newattcustomer.rar)

