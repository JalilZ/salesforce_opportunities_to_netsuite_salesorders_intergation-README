# Workato Recipe

Auto create netsuite sales order once the salesforce netsuite sync checkbox (in opportunity object) is checked</br>
For security measures, I cannot share the .json for this recipe - let me know if you need guidance on this topic !

## Workato actions explained

1. Trigger if netsuite sync checkbox (in opportunity object) is checked and retreive the triggered opportunity fields
2. Search if a netsuite sales order with the same opportunity ID already exists
3. get the salesforce opportunity's final quote ID (quote that the customer signed)
4. Get the End User customer SFDC id
5. Check if Fullfillment Partner (Reseller) is present, if yes then then the Reseller customer SFDC
6. Check if list from step 2 is greater that 0, if true then a sales order was already created for this opportunity, post a slack message detailing full data of this sales order (sales order number & link, opportunity ID, the end user customer, the reseller (if exists)) and Stop Job, if list is zero - continue recipe
10.  Create variables subsidairy_id, mp_customer_id, deal_type_id, reseller_id, enduser_id, payment_terms_inernalid, so_country_bill_to, so_state_bill_to, so_addressee_bill_to, so_address1_bill_to, so_city_bill_to, so_zip_bill_to, so_country_ship_to, so_state_ship_to, so_addressee_ship_to, so_address1_ship_to, so_city_ship_to, so_zip_ship_to, service_period_integer_months, billing_schedule

11. Check the quote payment terms and set the netsuite payment term id accordingly in the payment_terms_internalid variable (ruby)
12. Get subscription Terms months (rounded)
13. Determine the applicable netsuite billing schedule id according to the subscriptiom term and payment terms (ruby)
14. Get the subsidiary the customer signed the quote with, and determine the applicable netsuite subsidiary id
15. Check if CSP partner is present (marketplace deal), 
16. if step 15 is true then marketplace deals typically go only throught the corporate US's subsidiary, in that case over-ride the subsidary_id from step 14 and set the US subsidiary, and set the netsuite marketplace customer internal id per the CSP partner value (AWS/GCP/Azure) and set the deal_type as indirect and billing schedule as one-time payment (depends on company's policy with regards to marketplace deals)
17. Get the Netsuite marketplace customer record and post a slack message that marketplace customers already exists (corporate works only with 3 marketplaces; hence in no case we need salesforce to create more marketplace customers)
19. if Step 15 is false then get the netsuite subsidary record per the netsuite subsidiary id from step 14
20. check If CSP Partner is not present and Fullfillment Partner is present
21. If step 20 is true then the deal was not throght marketplace, but deal_type_id should be 2 (indirect deal) because fullfillment partner (reseller) is involved
22. Search if reseller customer record exists in netsuite
23. Get reseller country from salesforce and convert it to enumerated country (netsuite country format) - python
24. Get reseller state from salesforce and convert it to enumerated state (netsuite state format) - python
25. check if reseller record already exists
26. If step 25 is true, get the netsuite employee's record of the current salesforce reseller account owner
26. If step 25 is true, get the reseller record and save his internal id
28. If step 25 is true, set the geography (per netsuite format) per country & state from step 23 & 24
29. Update netsuite reseller record: account owner and address (always update address per deal as this might affect tax rates in netsuite) and set the type as reseller
30. Post slack message that a reseller already exists, and post its address and details
32. If step 25 is false, get the sales account owner employee record from netsuite (search using the salesforce account owner email)
33. Set the geography (per netsuite format) per country & state from step 23 & 24
34. Create Customer reseller record in netsuite
36. Retrieve the newly created reseller data from netsuite
37. Post slack message that a reseller was created (legal name, link to reseller record, geography, type, relevant subsidiary, SFDC ID, contact email)
38. Search end user customer record in netsuite
39. Get end user customer country Bill To from salesforce and convert it to enumerated country (netsuite country format) - python
40. Get end user customer country Ship To from salesforce and convert it to enumerated country (netsuite country format) - python
41. Get end user customer state Bill To  from salesforce and convert it to enumerated state (netsuite state format) - python
42. Get end user customer state Ship To  from salesforce and convert it to enumerated state (netsuite state format) - python
43. Check if end user customer record exists in netsuite
44. If step 43 is true, get the netsuite employee's record of the current salesforce end account owner
46. If step 43 is true, set the geography (per netsuite format) per country SHIP To & state SHIP To from step 41 & 42
47. Update netsuite end user record: account owner and address (always update address per deal as this might affect tax rates in netsuite) and set the type as end user
48. Post slack message that an end user already exists, and post its address and details
50. If step 43 is false, get the sales account owner employee record from netsuite (search using the salesforce account owner email)
51. If step 43 is false, set the geography (per netsuite format) per country SHIP To & state SHIP To from step 41 & 42
52. Create Customer end user record in netsuite
54. Retrieve the newly created end user data from netsuite
55. Post slack message that an end user was created (legal name, link to reseller record, geography, type, relevant subsidiary, SFDC ID, contact email)
56. For the Quote from step 3, get all quote lines (item lines)
57. Get the employee record of the sales rep that closed the opportunity (per opportunity owner email)
58. Using ruby, we set values for so_country_bill_to, so_state_bill_to, so_addressee_bill_to, so_address1_bill_to, so_city_bill_to, so_zip_bill_to, so_country_ship_to, so_state_ship_to, so_addressee_ship_to, so_address1_ship_to, so_city_ship_to, so_zip_ship_to. The conditions were set per the company's policy and wether a reseller was involved (depending also on the reseller salesforce type if MSSP or not) and other conditions that determines who is the "Bill To" entity and the "Ship To" entity and address
59. Get the End User Tax code from netsuite
60. Create a netsuite sales order: source quote lines and convert salesforce product codes to netsuite items internal ID's, get quantity, get rate, get billing schedule from step 10, get Tax Code from step 59, get start & end date & free months from each salesforce quote line, get order type from quote, determine deal type (ruby), get close-won date and sales rel internal id from step 57, set Bill to Country & address & addr1 & City & state & Zip, and set Ship To Country & address & addr1 & City & state & Zip, get payment terms from step 10 and set end customer customer from variable in step 10, get PO number from salesforce opportunity (nullify the value if CSP partner is present, no need for PO's in this case), set avatax Bill to and Ship to use tax codes depending on country, and set marketplace offer ID (if deal through MP)
61. Retrieve the newly created netsuite sales order data
62. Post a detailed slack message of the sales order that was created (opportunity name and ID, sales order reference & link, end user name & reseller (if applicable), geography, deal type, amount, taxes)

## Salesforce

Create a flow in salesforce that sends an outbound message to workato once the "Netsuite Sync" checkbox is checked (if the value was changed from false to true)

## Author

Jalil

