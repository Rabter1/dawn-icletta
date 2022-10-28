# Dawn
Dawn theme of shopify but with changes for ICLETTA

## Changes to theme: 

#### Rich Text
Added an option that allows the content width to make use of 100% of the available width

#### Product-recommendations
Added setting to 
 - input own product list
 - set a second heading for own product list

If a own product list is added and there are products inside, the second headding is used and the products are rendered. Otherwise the standard recommended products are shown.
Also some CSS changes
### Active workarounds:
Check, if these workarounds are still necesarry

#### 1) rich-text
It is not possible to use product description in rich text. So we pass `[product_description]` as a string and replace it in liquid code with the real deal.


#### 2) Translation of Metafield
Shopify does not support translations of metafields of type list
Shopify does not support translations of metafield labels
https://help.shopify.com/de/manual/markets/languages/translate-adapt-app#part-6ff81a217958e491

But these labels are used for filtering in frontend and need to be translated.
So as a workaround the name of these labels are passed into the translation. The key value pairs are added to the language json files.
File: facets.liquid