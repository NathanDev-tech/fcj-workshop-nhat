---
title : "End-to-End Testing"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.5.6 </b> "
---

After completing this section, the system will be verified to confirm:

- The entire user flow from Frontend through ALB to Backend (ECS) works correctly
- Storage and CDN services (S3, CloudFront, RDS MySQL) serve data accurately
- No errors, broken links, or data issues exist in the core features
- Serverless integrations and the Bedrock demo operate stably

### 5.5.6.1. Test Scenarios

#### Scenario 1. Registration → Login Flow

| Item | Details |
|------|---------|
| **Objective** | Ensure the account creation and login flow works successfully |
| **Expected Result** | User creates an account successfully, is able to log in, and the system updates the session state correctly |

**Steps:**

1. Access the website via CloudFront / ALB.
2. Select **Register** and fill in the form completely.
3. Use the newly created account to **Log in**.
4. Verify that the user information is displayed correctly after login.

**Edge Cases:**

- Case 1: Invalid email format or password too short → form shows validation error.
- Case 2: Login fails due to incorrect password.
- Case 3: Account already exists → system notifies duplicate email/username.

#### Scenario 2. Browse Books and View Product Details

| Item | Details |
|------|---------|
| **Objective** | Test the ability to load the book list and retrieve detailed information from the backend + RDS |
| **Expected Result** | Book list displays completely, images load from S3 Media, descriptions and prices are accurate |

**Steps:**

1. On the home page / category page, browse the book list.
2. Select a book to view the detail page.
3. Verify that the image, title, description, price, and related information are displayed correctly.

**Edge Cases:**

- Case 1: Image fails to load due to S3 Media connection error.
- Case 2: Book data is empty or RDS connection error.

#### Scenario 3. Add to Cart → Place Order + Apply Promo Code

| Item | Details |
|------|---------|
| **Objective** | Test the entire purchase flow and the promotional code feature |
| **Expected Result** | Book added to cart successfully, promo code applied correctly, order created successfully and status updated |

**Steps:**

1. Select a book → click **Add to Cart**.
2. Go to the cart, verify the quantity and total amount.
3. Apply a promo code (if available).
4. Proceed to checkout.
5. Verify the order is created and the status is displayed correctly (may receive notification via SNS).

**Edge Cases:**

- Case 1: Promo code is invalid or expired.
- Case 2: Order creation fails due to RDS connection loss.

#### Scenario 4. Image Upload

| Item | Details |
|------|---------|
| **Objective** | Ensure the image upload feature works and is processed by Lambda |
| **Expected Result** | Image is uploaded successfully to S3 Media, Lambda processes it (resize/optimize), and it displays correctly on the interface |

**Steps:**

1. Log in to an account.
2. Perform an image upload (book image / avatar / product-related image).
3. Verify the image appears on the interface after uploading.
4. Verify the file is saved on the S3 Media Bucket.

**Edge Cases:**

- Case 1: Upload a file with an invalid format (not an image).
- Case 2: File is too large → system rejects it.
- Case 3: Lambda image processing fails (original file is still saved).

#### Scenario 5. Admin Features

| Item | Details |
|------|---------|
| **Objective** | Test basic admin management functions |
| **Expected Result** | Admin can view and manage books, orders, and users normally |

**Steps:**

1. Log in with an Admin account.
2. Check the book list, add/edit/delete books (if applicable).
3. View the order list and update statuses.
4. Review user information.

**Edge Cases:**

- Case 1: A regular account does not have permission to access the Admin panel.
- Case 2: Data update fails due to database connection loss.

#### Scenario 6. Book Recommendation Demo with Amazon Bedrock

| Item | Details |
|------|---------|
| **Objective** | Test the book recommendation feature via Bedrock |
| **Expected Result** | The system returns a reasonable list of book recommendations based on user input |

**Steps:**

1. Access the book recommendation feature.
2. Enter a keyword / preference.
3. Verify the recommendation results returned from Bedrock (via Lambda / API Gateway).

**Edge Cases:**

- Case 1: Bedrock timeout or IAM permission error.
- Case 2: API throttling limit reached.
- Case 3: No matching results found.

#### Scenario 7. Error Handling Test

| Item | Details |
|------|---------|
| **Objective** | Ensure user-friendly exception handling mechanisms are in place |
| **Expected Result** | The interface displays clear error messages without crashing the application |

**Steps:**

1. Enter invalid data into forms (registration, order placement, etc.).
2. Simulate network latency or timeout scenarios.
3. Verify that error messages are displayed correctly and the system continues to function normally.

#### Scenario 8. Cross-Platform UI Test (Mobile Responsiveness)

| Item | Details |
|------|---------|
| **Objective** | Ensure consistent behavior on both Desktop and Mobile |
| **Expected Result** | The interface scales flexibly with no overflow or hidden critical components |

**Steps:**

1. Use browser Developer Tools (or a physical device) to simulate various screen sizes.
2. Check UI/UX on both mobile and desktop (home page, book details, cart, login form, etc.).

### 5.5.6.2. Testing Summary

The majority of test scenarios performed as expected. Edge cases were recorded, and most have been resolved or have fallback plans in place.

| Scenario                              | Result  |
|---------------------------------------|---------|
| Registration → Login                  | Passed  |
| Browse Books + View Details           | Passed  |
| Add to Cart → Place Order + Promo     | Passed  |
| Image Upload                          | Passed  |
| Admin Features                        | Passed  |
| Bedrock Book Recommendation Demo      | Passed  |
| Error Handling and Timeout            | Passed  |
| Mobile Responsiveness                 | Passed  |
