# Food Delivery Application (zomato/swiggy)
- online platform that allow user to serach restaurants and order food online from them.

## 1. Functional requirments
- user should be able to register into the application
- user should list down all the nearby restaurants based on their current location
- user should able to search restaurants based on title and menu.
- shows all menu in the display of the restaurants
- user should able to select various item the cart and make payment to confirm the order
- once the restaurants accepts the order, find nearby delivery partner based on driver location
- driver picked the order, give almost real time location of driver to user
- user should get all notification on all stage and get past orders to their profile

## 1.1 Non Functional requirments
- scale: 50M user 1M restuarants
- application should be highly availabile based on searching, should be highly consistent based on payments and order of food from restaurants 

## Core Entity
- user
- restaurants
- delivery partner

## API
###  register into the application (login/logout)
```
    POST: /v1/user/register {postbody: userMetadata}
```

###  list down all the nearby restaurants based on their current location
```
    GET: v1/restaurants/nearby?lat=${lat}&long=${long}&rad=${radius} 
    => List<restaurantsId (Partial)> :Pagination
```

### search restaurants based on title and menu
```
    GET: v1/restaurants/search?title=${title}&menuitems=${items} => List<restaurantsId (Partial)> :Pagination

```
### display of the restaurants with all menu
```
    GET: v1/restaurants/${id} 
    => restaurants metadata + review

    GET: v1/restaurants/${id}/menu 
    => List<items>
```
### Select various items into the cart and make order

```
    POST: /api/cart/items {postbody: itemId + quantity} + CRUD of items 
    => return orderId

    POST: /api/order {postbody: orderId} 
    => return orderId

    GET: api/delivery/${orderId}/tracking

```