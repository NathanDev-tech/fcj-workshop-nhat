---
title : "End-to-End Testing"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6.6 </b> "
---

After completing this section, the system will be tested and verified as follows:

- The entire user journey from the frontend through the ALB to the backend (ECS) works correctly.
- Data and images are displayed on the website through CloudFront.
- Core customer and administrator features work within the scope of testing.
- The serverless integration and Bedrock demo return responses through the user interface.

**Test environment:** [RakiBookery on CloudFront](https://d3nvhtmwuhmywh.cloudfront.net/)

### 5.6.6.1. Test scenarios

#### Scenario 1. Registration → Login flow

| Item | Details |
|-----|----------|
| **Objective** | Ensure successful account creation and login |
| **Expected result** | Users can successfully create an account and log in, and the system correctly updates the session status |

**Procedure:**

1. Access the website through CloudFront.
2. Select **Login** and enter the registered customer account credentials.
3. Click **Login** and verify the session status.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/12-login-form.png)

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/13-login-success.png)

4. Check the personal account, shopping cart, orders and logout menu entries.

**Exception cases:**

- Case 1: Invalid email format or short password → the form displays validation errors.
- Case 2: Login fails → the application reports incorrect email or password.
- Case 3: An account already exists → verify the duplicate-email notification.

#### Scenario 2. Browse books and view product details

| Item | Details |
|-----|----------|
| **Objective** | Verify the ability to load the catalog and retrieve product details from the Backend and RDS |
| **Expected result** | The complete book list is displayed, with accurate images, book titles, descriptions and prices |

**Procedure:**

1. Open the book list on the home page or catalog page and verify the 20 products.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/02-shop.jpg)

2. Select a book to open its product detail page.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/03-product-detail.jpg)

3. Check the cover image, book title, author, publisher, description and price.

**Exception cases:**

- Case 1: An image fails to load due to an S3 Media connection issue.
- Case 2: Book data is empty or the RDS connection fails.

#### Scenario 3. Add to cart → Checkout + Apply promotions (Promo)

| Item | Details |
|-----|----------|
| **Objective** | Test the complete purchasing flow, total calculations, payment and promotion codes |
| **Expected result** | Books are successfully added to the cart, promotion codes are correctly applied, orders are successfully created and their statuses are updated |

**Procedure:**

1. Select a book and click **Add to cart**.
2. Open the cart and verify quantity and totals. Two copies of Harry Potter total VND 360,000.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/04-cart.jpg)

3. Apply a promotion code if available. The input field is available, but the result of applying a code remains unverified.
4. Proceed to place the order. Test PayPal Sandbox with an order of two books worth VND 240,000. The payment window displays USD 9.60.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/14-paypal-sandbox.png)

5. Verify the **Order placed successfully** notification.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/15-order-success.png)

6. Select **View order**. Cross-check order number 75, its two products, the VND 240,000 total, the PayPal payment method and the Pending confirmation status.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/16-order-detail.png)


**Exception cases:**

- Case 1: Invalid or expired promotion code.
- Case 2: Order creation fails due to an RDS connection issue.
- Case 3: The user is not logged in → the system requires login before payment.

#### Scenario 4. Image upload

| Item | Details |
|-----|----------|
| **Objective** | Ensure that image upload works, images are stored and processed by Lambda, and images are displayed correctly on the website |
| **Expected result** | Images are uploaded to S3 Media, processed by Lambda and displayed correctly on the user interface. If a product has no image or its image fails to load, the system displays an appropriate placeholder |

**Procedure:**

1. Sign in using an account authorized to upload images.
2. Upload a book image or profile image.
3. Check that the image is displayed on the catalog or product detail page.
4. Check the file in the S3 Media Bucket and the Lambda logs.

**Exception cases:**

- Case 1: Unsupported upload file format.
- Case 2: Oversized file → the application rejects the upload.
- Case 3: Lambda image processing fails.
- Case 4: A product has no image. When checking the test product `abc`, the interface displays the default **No Product Image Available** placeholder.

  ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/23-upload-image-placeholder.jpg)

- Case 5: A product image fails to load. When checking **The Song of Achilles**, the cover image is not displayed and the browser displays alternative text. The adjacent product's cover image still loads correctly.

  ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/24-upload-image-load-error.jpg)

The test confirms that the interface displays a default image for products without images and identifies one instance of a cover image failing to load. The image URL, S3 Media object and Lambda logs must be checked to establish the cause and complete verification of the upload workflow.

#### Scenario 5. Admin functionality

| Item | Details |
|-----|----------|
| **Objective** | Test basic administrative functions using an administrator account |
| **Expected result** | Administrators can access and view the dashboard, products, categories, orders, users and promotions |

**Procedure:**

1. Sign in as an administrator and open **Dashboard**. Check the order count, revenue, user count, product count and statistical charts.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/17-admin-dashboard.png)

2. Open **Product Management** and check the book list, images, statuses and action buttons.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/18-admin-products.png)

3. Open **Category Management** and verify the five book categories.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/19-admin-categories.png)

4. Open **Order Management**. Check the order list. Order number 75 has a total of VND 240,000, the PayPal payment method and the Pending confirmation status.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/20-admin-orders.png)

5. Open **User Management**. Verify the user list, Customer/Admin roles and account statuses.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/21-admin-users.png)

6. Open **Promotion Management**. Verify three promotion codes with 10%, 20% and 30% discounts.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/22-admin-promotions.png)


**Exception cases:**

- Case 1: A regular account does not have permission to access administration pages.
- Case 2: Data updates fail due to a lost database connection.
- Case 3: Adding, editing and deleting records, locking accounts and changing order statuses require verification using test data.

#### Scenario 6. Amazon Bedrock book recommendation demo

| Item | Details |
|-----|----------|
| **Objective** | Test the book recommendation functionality through Bedrock |
| **Expected result** | The system returns book recommendations relevant to the user's input |

**Procedure:**

1. Open the book recommendation feature.
2. Enter a question asking for a recommendation for a book about artificial intelligence available at RakiBookery.
3. Verify the response: Dẫn nhập ngắn về khoa học - Trí tuệ nhân tạo, VND 180,000, with 24 copies available.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/06-chatbot.jpg)


**Exception cases:**

- Case 1: Bedrock timeout or IAM authorization error.
- Case 2: API throttling limit reached.
- Case 3: No matching recommendations.

#### Scenario 7. Error handling

| Item | Details |
|-----|----------|
| **Objective** | Ensure user-friendly exception handling |
| **Expected result** | The interface displays clear error messages and the application continues to operate |

**Procedure:**

1. Enter invalid data in the registration form and verify validation messages.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/07-registration-validation.jpg)

2. Observe the error message during a rejected login attempt.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/01-login-failed.jpg)

3. Start checkout while unauthenticated and verify the warning.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/05-checkout-login-required.jpg)

4. Continue browsing the catalog and using the cart.

**Exception cases:**

- Case 1: Invalid input data.
- Case 2: Login failure or unauthenticated checkout.
- Case 3: Timeouts, network latency and connection interruptions must be tested in a controlled environment.

#### Scenario 8. Cross-platform UI testing (Mobile Responsiveness)

| Item | Details |
|-----|----------|
| **Objective** | Verify consistent behavior on desktop and mobile |
| **Expected result** | The interface adapts to screen size and essential elements remain accessible |

**Procedure:**

1. Use browser Developer Tools to emulate a mobile viewport of 390 × 844 px.
2. Check the home page interface.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/08-mobile-home.jpg)

3. Check the book detail page.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/09-mobile-detail.jpg)

4. Check the shopping cart, total amount and horizontal scrolling functionality.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/10-mobile-cart.jpg)

5. Check the login form on mobile.

   ![](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.6-End-to-End-testing/11-mobile-login.jpg)


### 5.6.6.2. Test summary

Core customer journeys and access to the administration pages have been tested. Results are recorded based on the available evidence.

| Scenario | Result |
|---|---|
| Registration → Login flow | Passed  |
| Browse books and view product details | Passed |
| Add to cart → Checkout + Apply promotions (Promo) | Passed|
| Image upload | Passed |
| Admin functionality | Passed (access to and display of administration pages) |
| Amazon Bedrock book recommendation demo | Passed |
| Error handling | Passed |
| Cross-platform UI testing (Mobile Responsiveness) | Passed |

In total, six scenarios passed within the tested scope and two scenarios partially passed.
