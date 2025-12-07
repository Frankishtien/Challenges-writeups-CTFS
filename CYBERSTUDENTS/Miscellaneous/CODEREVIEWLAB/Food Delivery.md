# Food Delivery


#### Understanding the Vulnerability

This application has a critical information disclosure vulnerability in the restaurant details API endpoint. The vulnerability exists in

```
api/restaurants.php
```

where the

```
getRestaurantDetails
```

function retrieves restaurant information along with all orders associated with that restaurant, regardless of which user is making the request.

The problematic code fetches orders using only the

```
restaurant_id
```

as a filter without checking if those orders belong to the authenticated user:

```
$orders = $db->query("SELECT * FROM orders WHERE restaurant_id = {$restaurant_id}");
```


This means any authenticated user can view complete order histories from other customers including:

-   Customer names and addresses
-   Phone numbers
-   Order contents and preferences
-   Delivery instructions
-   Payment information
-   Order timestamps

An attacker could systematically enumerate all restaurants and collect sensitive personal information about thousands of users. This violates user privacy, GDPR/data protection regulations, and could lead to identity theft, stalking, or targeted phishing attacks.

The authentication middleware correctly validates the user's session, but the authorization logic fails to restrict data access to only the current user's orders.




