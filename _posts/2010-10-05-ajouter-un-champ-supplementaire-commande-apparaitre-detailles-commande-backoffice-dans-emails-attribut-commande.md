---
layout: post
title:  "Ajouter un champ supplémentaire dans la commande - attribut commande"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/champ-supplementaire-ommande.jpg
---
Supposant qu’on aime avoir une information supplémentaire sur la commande de notre client par exemple : “Heur de livraison souhaitée”, “au nom de”, commentaire….
Cette information qui doit être bien sur affiché dans les détailles de la commande et qu’on veut envoyer aussi pas les emails…
Aujourd’hui on va crée un petit module qui gère ca.
La structure de notre petit module :

![attribut commande magento]({{ site.baseurl }}/assets/images/module_magento2.jpg)


Donc la première chose à faire est de penser comment stocker cette information
dans la BD Magento :
On va utiliser un attributs ![:)](http://blog.ahmedmahi.com/wp-includes/images/smilies/icon_smile.gif) (heur_livraison)!
Dans notre fichier mysql4-install-0.1.0.php on va ajouter :

```php
<?php
$installer = $this;
$installer->startSetup();
$setup = new Mage_Eav_Model_Entity_Setup('core_setup');
 
 
$installer->getConnection()->addColumn(
        $installer->getTable('sales_flat_order'),
        'heur_livraison',
        'varchar(255) DEFAULT NULL AFTER `shipping_method`'
);
$setup->startSetup();
$setup->addAttribute('order', 'heur_livraison', array('type' => 'text'));
$installer->endSetup();
```

Vous remarquez que ce n’est pas assez simple comme pour [les attributs clients](http://test.mahigento.com/ajouter-des-champs-specifiques-dans-linscription-dun-nouveau-client-magento), pour les commandes on a obligé d’ajouter une ligne dans la table «sales_flat_order» ou utiliser addAttribute des deux class Mage_Sales_Model_Mysql4_Setup et Mage_Sales_Model_Entity_Setup.
Donc cela va crée un attribut « commande ».
Maintenant je passe à la façon d’enregistrer la valeur de cet attribut.
Puisque c’est information déponde du mode de livraison donc je vais ajouter mon champ à la fin du fichier : checkout/ shipping_method/ available.phtml

```html
<div class="field">
    <label for="heur_livraison_souhaitee"><strong>Heur de livraison souhaitée</strong></label>
    <div class="input-box">
        <input type="text" name="heur_livraison_souhaitee" id="heur_livraison_souhaitee"  class="input-text validation-passed" />
    </div>
</div>
```

Résultat dans onepage :

![attribut heure livraison commande]({{ site.baseurl }}/assets/images/magento_heure_livraison.jpg)

L’action concernée est saveShippingMethod de la class Mage_Checkout_OnepageController :
Donc on peut surcharger la méthode ce que je ne préfère pas comme je dit toujours ![:)](http://blog.ahmedmahi.com/wp-includes/images/smilies/icon_smile.gif) ou utiliser un observer, puisque il s’agit d’une action je vais utiliser l’event controller_action_postdispatch c’est mon préféré :).
Est donc dans le fichier config.xml :

```xml
<events>
            <controller_action_postdispatch>
                <observers>
                    <mahigento_addattorder_gethls>
                        <type>singleton</type>
                        <class>addattorder/observer</class>
                        <method>getHeurLivraison</method>
                    </mahigento_addattorder_gethls>
                </observers>
            </controller_action_postdispatch>
            <sales_order_place_before>
                <observers>
                    <mahigento_addattorder_sethls>
                        <type>singleton</type>
                        <class>addattorder/observer</class>
                        <method>setHeurLivraison</method>
                    </mahigento_addattorder_sethls>
                </observers>
            </sales_order_place_before>
        </events>
```

Il y’a en fait 2 observer que je vais utiliser car le premier de l’event controller_action_postdispatch c’est juste pour conservé la valeur du champ dans la session c’est le deuxième qu’on va utiliser pour l’enregistrement dans la commande
Les 2 observers sont lié au 2 méthodes de mon fichier Observer.php suivants :



```php
<?php
 
class Mahigento_Addattorder_Model_Observer {
 
    public function getHeurLivraison($observer) {
 
        $Controller = $observer->getControllerAction();
        if ($Controller instanceof Mage_Checkout_OnepageController) {
            $actionName = $Controller->getFullActionName();
            if ($actionName == 'checkout_onepage_saveShippingMethod') {
                $data = $Controller->getRequest()->getPost();
                if (isset($data['heur_livraison'])) {
                    Mage::getSingleton('core/session')->setHeurLivraison($data['heur_livraison']);
                }
            }
        }
    }
 
    public function setHeurLivraison($observer) {
        $order = $observer->getOrder();
        $session = Mage::getSingleton('core/session');
        if ($session->hasHeurLivraison()) {
            $order->setData('heur_livraison', $session->getHeurLivraison());
            $session->unsetHeurLivraison();
        }
    }
 
}
```

Le rôle de la première méthode est d’enregistré la valeur du champ si elle existe dans la session en attendant qu’avant directement le passage de la commande « sales_order_place_before » la méthode setHeurLivraison affecte la valeur de la session vers l’attribut commande.
Et enfin pour afficher la valeur dans les détails de la commande dans le backoffice on a qu’ajouter par exemple dans :
Design/adminhtml/default/default/template/sales/order/view/tab/info.phtml (ligne 79).

```php+HTML
<br />
<?php echo $this->__('Heure de livraison souhaitée :'). $_order-
>getHeurLivraison() ?>
```

Résultat :

![attribut commande magento]({{ site.baseurl }}/assets/images/information_livraison_magento_mahigento.jpg)

Et pour les emails on peut utiliser dans les gabaries la variable

> {  {var order.heur_livraison }  }

Noubliez pas de vider le cache et se déconnecté puis faites votre première commande test :).

Testé sur 1.4.1.0 et 1.4.1.1