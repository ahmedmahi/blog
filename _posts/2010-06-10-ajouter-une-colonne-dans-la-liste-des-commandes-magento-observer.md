---
layout: post
title:  "Ajouter une colonne dans la liste des commandes Magento (observer)"
author: ahmed
categories: [ magento, tutorial ]
image: assets/images/colonne-liste-commandes-magento.jpg
---
Aujourd’hui on va essayer d’ajouter une colonne dans la liste des commandes mais on ne va pas surcharger la classe :

> Mage_Adminhtml_Block_Sales_Order_Grid

Puisque ce n’est pas recommandée dans Magento : quand une même classe est surchargée par 2 module cela cause un conflit, des fois bien sur on n’a pas le choix, contrairement a notre cas d’aujourd’hui ![:)]({{ site.baseurl }}/assets/images/icon_smile.gif)

Alors on va utiliser un observer : l’événement qui nous intéresse est

> adminhtml_block_html_before  

Donc on doit ajouter dans notre fichier config.xml :

```xml
<?xml version="1.0"?>
<config>
    <modules>
        <Mahigento_Newcolumn>
            <version>0.1.0</version>
        </Mahigento_Newcolumn>
    </modules>
    <global>
        <models>
            <newcolumn>
                <class>Mahigento_Newcolumn_Model</class>
            </newcolumn>
        </models>
        <events>
	<adminhtml_block_html_before>
                <observers>
                    <mahigento_newcolumn_grid>
                        <type>singleton</type>
                        <class>newcolumn/grid</class>
                        <method>ajouter</method>
                    </mahigento_newcolumn_grid>
                </observers>
        </adminhtml_block_html_before>
        </events>
	</global>
</config>
```

Comme vous remarqué ici on a déclaré le nom de la classe et le nom de la méthode à exécuter lorsque l’événement se produit, attaquons donc la méthode ‘ajouter’ de la classe 

> Mahigento_Newcolumn_Model_Grid (Grid.php)

```php
<?php
class Mahigento_Newcolumn_Model_Grid {
    public function ajouter($observer) {
        $block = $observer->getEvent()->getBlock();
        if($block instanceof Mage_Adminhtml_Block_Sales_Order_Grid){
            $block->addColumn('shipping_description',array(
            'header' => Mage::helper('sales')->__('Methode de livraison'),
            'index' => 'shipping_description',
            'type' => 'text',
            'width' => '100px',
             ),
            'real_order_id');


            }
        }
    }
 
}
```

Alors cette fonction va ajouter une colonne ‘Méthode de livraison’ pour voire directement de quel mode de livraison s’agit- il sont avoir besoin d’ouvrir la commande, si on veut ajouter une autre information par exemple le mode de paiement
On va juste modifier la méthode addColumn :

```php
$block->addColumn('payment_method',array(
            'header' => Mage::helper('sales')->__('Méthode de paiement'),
            'index' => 'payment_method',
            'type' => 'text',
            'width' => '100px',
             ));
```

La seul contrainte c’est que cette information doit être déjà dans la collection des commandes.
Si c’est pas le cas par exemple je veux ajouter l’email et le groupe client je doit alors modifier la collection :
Pour cela je vais ajouter la méthode:

```php
 public function  getnewcollection (){
        $collection = Mage::getResourceModel('sales/order_grid_collection');
        $collection->getSelect()
          ->join(array('client' => $collection->getTable('customer/entity')),
              'client.entity_id = main_table.customer_id',
              array('email','group_id'))
        ->join(
            array('order'=>$collection->getTable('sales/order')),
                'order.entity_id=main_table.entity_id',
            array('shipping_description'));
    
        return $collection;
    }
```

et puis :

> $block->setCollection($this->getnewcollection());//affecter la nouvelle collection 

maintenant pour rendre mes nouvelles colonnes filtrables et tri-ables j’utilise ça :

```php
    public function  FilterAndSort ($block,$newcolumns){
        $filter=$block->getParam($block->getVarNameFilter(), null);
        $data = Mage::helper('adminhtml')->prepareFilterString($filter);
        $columns=$block->getColumns();
        foreach($columns as $columnId => $column){
            $field = ( $column->getFilterIndex() ) ? $column->getFilterIndex()
           : $column->getIndex();
            if(in_array($columnId, $newcolumns)){
                if (isset($data[$columnId]) && (!empty($data[$columnId]) ||
                    strlen($data[$columnId]) > 0)
               && $column->getFilter()) {
                    $column->getFilter()->setValue($data[$columnId]);
                    $cond = $column->getFilter()->getCondition();
                    if ($field && isset($cond)) {
                     $block->getCollection()->addFieldToFilter($field , $cond);
                    }
                }
                $columnIdNameSort = $block->getParam($block->getVarNameSort(),
                                   $block->_defaultSort);
                if(in_array($columnIdNameSort, $newcolumns)){
                    $dir= $block->getParam($block->getVarNameDir(),
                                 $block->_defaultDir);
                    $dir = (strtolower($dir)=='desc') ? 'desc' : 'asc';
                    $column_s=$columns[$columnIdNameSort];
                    $column_s->setDir($dir);
                    $column_s = $column_s->getFilterIndex() ?
                    $column_s->getFilterIndex() : $column_s->getIndex();
                    $block->getCollection()->setOrder($column_s , $dir);
                }
            }
        }
    }
```

et pour utiliser cette fonction :

> $this->FilterAndSort($block,array('email','group','shipping_description'));

Donc finalement notre class : Mahigento_Newcolumn_Model_Grid devient:

```php
<?php
class Mahigento_Newcolumn_Model_Grid  {

    public function ajouter($observer) {
        $block = $observer->getEvent()->getBlock();
        if ($block instanceof Mage_Adminhtml_Block_Sales_Order_Grid) {
            $block->addColumn('shipping_description',array(
            'header' => Mage::helper('sales')->__('Methode de livraison'),
            'index' => 'shipping_description',
            'type' => 'text',
            'width' => '100px',
                ));
            $groups = Mage::getResourceModel('customer/group_collection')
            ->addFieldToFilter('customer_group_id', array('gt'=> 0))
            ->load()
            ->toOptionHash();

            $block->addColumn('group', array(
            'header'    =>  Mage::helper('customer')->__('Group'),
            'index'     =>  'group_id',
            'type'      =>  'options',
            'width'     =>  '50px',
            'options'   =>  $groups,
                ));
            $block->addColumn('email', array(
                'header' => Mage::helper('sales')->__('email'),
                'index' => 'email',
                'type' => 'text',
                'width' => '50px',
                ));

    $block->setCollection($this->getnewcollection());
    $this->FilterAndSort($block,array('email','group','shipping_description'));

        }
    }
    public function  FilterAndSort ($block,$newcolumns){
        .......
    }
    public function  getnewcollection (){
            .......
          return $collection;
    }
}
```

[Télécharger les fichiers]({{ site.baseurl }}/assets/files/Newcolumn.rar)