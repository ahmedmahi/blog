---
layout: post
title:  "Rendre le message cadeau payant – magento"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/message-cadeau.jpg
featured: true
hidden: true
---

Aujourd’hui je vous propose un petit module que j’ai développé pour rendre le message cadeau payant l’idée est d’ajouter un produit “Message cadeau” au panier si le message est spécifié lors du passage de la commande.
Au niveau utilisation c’est très simple il faut d’abord activer les messages cadeau :
System -> configuration -> Ventes -> Messages cadeau-> Autoriser les messages cadeau au niveau de la commande.
Puis crée le produit “Message cadeau” et n’oubliez pas de mettre une quantité positive du stock ou de choisir “Non” pour “Gérer les stocks” sinon le produit ne s’ajoute pas.
Pius installer le module, allez à System -> configuration -> Ventes -> Produit message cadeau pour activer le module et ajouter l’ID du produit.
Au niveau développement ![:)]({{ site.baseurl }}/assets/images/icon_smile.gif) :
Il s’agit d’un observer de deux events :
checkout_controller_onepage_save_shipping_method et checkout_controller_multishipping_shipping_post (pour le choix du multishipping ) donc un fichier config.xml qui correspond à :

```xml
<?xml version="1.0"?>
<!-->
/**
 * @category   Mahigento
 * @package    Mahigento_Pgm
 * @author    Ahmed MAHI/ S3i Business: 1hmedmahi@gmail.com -www.ahmedmahi.com
 * @license   http://opensource.org/licenses/osl-3.0.php  Open Software License (OSL 3.0)
 */
 <-->
<config>
    <modules>
        <Mahigento_Pgm>
            <version>0.1.0</version>
        </Mahigento_Pgm>
    </modules>
        <adminhtml>
        <acl>
            <resources>
                <admin>
                    <children>
                        <system>
                            <children>
                                <config>
                                    <children>
                                        <pgm translate="title" module="pgm">
                                       <title>Produit message cadeau</title>
                                            <sort_order>50</sort_order>
                                        </pgm>
                                    </children>
                                </config>
                            </children>
                        </system>
                    </children>
                </admin>
            </resources>
        </acl>
    </adminhtml>
    <global>
        <models>
            <pgm>
                <class>Mahigento_Pgm_Model</class>
            </pgm>
        </models>
        <events>
            <checkout_controller_onepage_save_shipping_method>
                <observers>
                    <mahigento_pgm_shipping>
                        <type>model</type>
                        <class>pgm/pgm</class>
                        <method>addGiftMessageProduct</method>
                    </mahigento_pgm_shipping>
                </observers>
            </checkout_controller_onepage_save_shipping_method>
            <checkout_controller_multishipping_shipping_post>
                <observers>
                    <mahigento_pgm_multishipping>
                        <type>model</type>
                        <class>pgm/pgm</class>
                        <method>addGiftMessageProduct</method>
                    </mahigento_pgm_multishipping>
                </observers>
            </checkout_controller_multishipping_shipping_post>
        </events>
    </global>
</config>
```

Et un fichier system.xml pour ajouter les éléments de la configuration :activation du module et la spécification de l’id du produit à ajouter.

```xml
<?xml version="1.0"?>
<!-->
/**
 * @category   Mahigento
 * @package    Mahigento_Pgm
 * @author    Ahmed MAHI/ S3i Business: 1hmedmahi@gmail.com -www.ahmedmahi.com
 * @license   http://opensource.org/licenses/osl-3.0.php  Open Software License (OSL 3.0)
 */
 <-->
<config>
    <sections>
        <pgm translate="label" module="pgm">
            <label>Produit message cadeau</label>
            <tab>sales</tab>
            <frontend_type>text</frontend_type>
            <sort_order>900</sort_order>
            <show_in_default>1</show_in_default>
            <show_in_website>1</show_in_website>
            <show_in_store>0</show_in_store>
            <groups>
                <settings translate="label">
                    <label>Settings</label>
                    <frontend_type>text</frontend_type>
                    <sort_order>1</sort_order>
                    <show_in_default>1</show_in_default>
                    <show_in_website>1</show_in_website>
                    <show_in_store>0</show_in_store>
                    <fields>
                        <enabled translate="label">
                            <label>Activer</label>
                            <frontend_type>select</frontend_type>
                            <source_model>adminhtml/system_config_source_yesno</source_model>
                            <sort_order>1</sort_order>
                            <show_in_default>1</show_in_default>
                            <show_in_website>1</show_in_website>
                            <show_in_store>0</show_in_store>
                        </enabled>
                        <productid translate="label">
                            <label>ID produit</label>
                            <frontend_type>text</frontend_type>
                            <sort_order>2</sort_order>
                            <show_in_default>1</show_in_default>
                            <show_in_website>1</show_in_website>
                            <show_in_store>0</show_in_store>
                        </productid>
                    </fields>
                </settings>
            </groups>
        </pgm>
    </sections>
</config>
```

Et l’observer avec quelques commentaires :

```php
<?php 
    /**  
    * @category   Mahigento   
    * @package    Mahigento_Pgm   
    * @author    Ahmed MAHI/ S3i Business: 1hmedmahi@gmail.com -www.ahmedmahi.com  
    * @license   http://opensource.org/licenses/osl-3.0.php  Open Software License (OSL 3.0)  
    */ 
    class Mahigento_Pgm_Model_Pgm  {     
    private $_giftId;     public function __construct() {         if(Mage::getStoreConfig('produitcadeau/settings/enabled')) {             
        $this->_giftId =Mage::getStoreConfig('pgm/settings/productid');
        }
    }
    // La méthode principale d'ajout de notre produit "message cadeau"
    public function addGiftMessageProduct($observer)
    {
       //Vérifier si le module est activé
        if(Mage::getStoreConfig('produitcadeau/settings/enabled')) {
            $giftMessages = $observer->getEvent()->getRequest()->getParam('giftmessage');
            if(is_array($giftMessages)) {
               // Supprimer le produit s'il est déjà dans le panier
               $this->removeGiftMessageProduct($observer->getQuote());
                if($product=$this->getProduct()) {
                    foreach ($giftMessages as $entityId=>$message) {
                      //Si le message est vide ne fait rien continué
                        if(trim($message['message'])=='') {
                            continue;
                        }
                        $cart=Mage::getSingleton('checkout/cart');
                        $cart->addProduct($product);
                        $cart->save();
                    }
                }
                //Sinon une fois un changement détecté dans le panier il aura une redirection vers la page panier (la méthode _expireAjax du controlleur)
                Mage::getSingleton('checkout/session')->setCartWasUpdated(false);
            }
            return $this;
        }}
    //Supprimer le produit du panier
    public function removeGiftMessageProduct($quote) {
        foreach ($quote->getAllItems() as $item) {
            if($item->getProductId() == $this->_giftId) {
                Mage::getSingleton('checkout/cart')->removeItem($item->getId())->save();
            }
        }
    }
    public function getProduct() {
        if ($this->_giftId) {
            $product = Mage::getModel('catalog/product')
            ->load($this->_giftId);
            Mage::log($product->getStockItem());
            //"En stock" ou  "Non" pour  "Gérer les stocks" sinon le produit ne pourra pas être ajouté
            if ($product->getId()&amp;&amp;(!$product->getStockItem()->getUseConfigManageStock()||$product->getStockItem()->getIsInStock())) {
                return $product;
            }
        }
        return false;
    }
}
```

