Getting Started With SiddharthCartBundle
========================================

## Prerequisites
First thing before using SiddharthCartBundle you have to create your own ProductBundle.

This version of the bundle requires Symfony 2.1.

For more information about Bundle, check [Symfony documentation](http://symfony.com).

## Create Your Own Cart Bundle

Please Follow Few Step:

1. Generate SiddharthCartBundle using command prompt
2. Create Your Cart Controller
3. Create your Cart View

### Step 1: Generate SiddharthCartBundle using command prompt:

``` bash
php app/console generate:bundle --namespace=Siddharth/CartBundle
```

### Step 2: Create Your Cart Controller:

Create-
`CartBundle/Controller/CartController.php`

``` php
<?php
  
namespace Siddharth\CartBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
use Symfony\Component\HttpFoundation\Response;

use Siddharth\ProductBundle\Entity\Product;

/**
 * @Route("/cart")
 */
class CartController extends Controller
{
	/**
     * @Route("/", name="cart")
     */
	public function indexAction()
	{
		
		$session = $this->getRequest()->getSession();
		$cart = $session->get('cart', array());
		
		if( $cart != '' ) {
			
			foreach( $cart as $id => $quantity ) {
				      $productIds[] = $id;
			
			}			
			
			if( isset( $productIds ) )
			{
				$em = $this->getDoctrine()->getEntityManager();
				$product = $em->getRepository('SiddharthProductBundle:Product')->findById( $productIds );
			} else {
				return $this->render('SiddharthCartBundle:Cart:index.html.twig', array(
					'empty' => true,
				));
			}
			
			return $this->render('SiddharthCartBundle:Cart:index.html.twig', array(
				'product' => $product,
			));
		} else {
			return $this->render('SiddharthCartBundle:Cart:index.html.twig', array(
				'empty' => true,
			));
		}
	}


	/**
     * @Route("/add/{id}", name="cart_add")
     */
	public function addAction($id)
	{
		$em = $this->getDoctrine()->getEntityManager();
		$product = $em->getRepository('SiddharthProductBundle:Product')->find($id);
		$session = $this->getRequest()->getSession();
	  $cart = $session->get('cart', array());
	    
		
		if ( $product == NULL ) {
			 $this->get('session')->setFlash('notice', 'This product is not available in Stores');			
			return $this->redirect($this->generateUrl('cart'));
		} else {
			if( isset($cart[$id]) ) {
				
				$qtyAvailable = $product->getQuantity();
				
				if( $qtyAvailable >= $cart[$id]['quantity'] + 1 ) {
					$cart[$id]['quantity'] = $cart[$id]['quantity'] + 1; 
				} else {
					$this->get('session')->setFlash('notice', 'Quantity exceeds the available stock');			
					return $this->redirect($this->generateUrl('cart'));
				}
			} else {
				$cart = $session->get('cart', array());
				$cart[$id] = $id;
				$cart[$id]['quantity'] = 1;
			}
			
			$session->set('cart', $cart);
			return $this->redirect($this->generateUrl('cart'));
			
		}
	}


	/**
     * @Route("/remove/{id}", name="cart_remove")
     */
	public function removeAction($id)
	{
		$session = $this->getRequest()->getSession();
	    $cart = $session->get('cart', array());
		
		if(!$cart) { $this->redirect( $this->generateUrl('cart') ); }
		
		if( isset($cart[$id]) ) {
			$cart[$id]['quantity'] = '0';
			unset($cart[$id]);
		} else {
			$this->get('session')->setFlash('notice', 'Go to hell');	
			return $this->redirect( $this->generateUrl('cart') );
		}
		
		$session->set('cart', $cart);
		
		$this->get('session')->setFlash('notice', 'This product is Remove');
		return $this->redirect( $this->generateUrl('cart') );
	}
}
```

### Step 3: Create your Cart View:

Create-
`Resources/views/Cart/index.html.twig`

``` php

{% block body %}
<h1>"Siddharth" SHOPPING-CART</h1>

<ul class="thumbnails">

{% if empty %}
    <h5>Your shopping cart is empty.</h5>
{% endif %}
{% set cart = app.session.get('cart') %}


{% if product %}
  
<ul class="thumbnails">
{% if app.session.hasFlash('notice') %} 

     <divclass="flash-notice">

      {{app.session.flash('notice') }} 
      {{ app.session.removeFlash('notice') }}
     
     </div>

{% endif %} 		
{% for key, item in cart %}
		<p>ID:{{ key }}</p>
		 <p>Quantity:{{ item }}</p>
		 <button class="btn btn-primary"><a href="{{ path('cart_remove', {'id': key}) }}">Remove</a></button>
{% endfor %}
</ul>

{% endif %}
</ul>

<a href="{{ path('products') }}">Products</a>

{% endblock %}
```