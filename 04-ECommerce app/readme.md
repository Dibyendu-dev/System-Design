# E-Commerce Application (Amazon/Myntra/Flipkart)
- online shopping platform where user can purchase electronics, books, dress, home appliances and many more.

## Functional requirments:
- user should be able to register into the application
- user should be able to search and finds product based on title description.
- user should able to view the details about the product (description, image, review, available quantity)
- user should  select the quantity and move the items into the cart
-  user should able to make the payment and do the checkout
- user should able to check the status of the order

## Non-Functional Requirments:
- scale: 10M MAU and 10 order/sec
- low latency: searching
- high availability w.r.t searching and viewing the items highly consistant w.r.t placing the order

## Core entity
- user
- product
- cart
- order
- checkout

## API Design

###  register into the application (login/logout)
```
    POST /v1/user/register
    POST /v1/user/login
    POST /v1/user/logout
```
### search and finds product
```
    GET /v1/products/search?q=iphone&page=1&limit=20&sort=price

```
### get product details
```
    GET /v1/product/{productId}
```

### add to cart
```
    POST   /v1/cart/items        // add item
    PUT    /v1/cart/items/{id}   // update qty
    DELETE /v1/cart/items/{id}
    GET    /v1/cart
```

### add to checkout + payment
```
    POST /v1/checkout          // validates cart, locks inventory, creates order
    POST /v1/payments          // interacts with payment gateway
```

### check order status
```
    GET /v1/orders/{orderId}
```
