---
title: Order refresh and processing
taxonomy:
    category: docs
---

Order processing is part of the order refresh process. This is run when on draft orders to ensure that it has up to date adjustments and that its order items are up to date.

## The order refresh lifecycle

The order refresh is vital to ensuring that an order has up to date product pricing, promotional adjustments, taxes, and more. When an order is loaded, the order's storage checks if it requires a refresh. This then invokes the order refresh cycle, controlled by the `commerce_order.order_refresh` process.

!!! The order refresh process only runs on **draft** orders. It will not run on orders which have been placed.

The following is an overview of the process:

* An order is loaded from storage and checked if it should be refreshed.
* All adjustments on the order are removed.
* Each order item is processed
  * All adjustments are removed.
  * If there is a purchasable entity reference, the order item's price is updated to that purchasable entity's current price.
* Services that have hooked into the process will run on the order
  * The Promotions module uses this to apply order and order item promotions adjustments.
  * Tax applies tax adjustments.
* If order items have been changed (new pricing, new adjustments) they are saved.

## Configure the order refresh threshold

Each order type can configure it's refresh threshold. For instance, a regular customer order type might be fine at an hourly refresh interval. On the other hand, a wholesaler order type may need a more constant refresh, such as every minute, due to pricing logic.

By default order types refresh every five minutes. This can be modified by editing the order type `/admin/commerce/config/order-types/ORDER-TYPE/edit`.

![Order refresh settings](images/order-type-order-refresh.png)

## Hooking into the process

The order refresh process uses tagged services to identify services which should be ran. Service classes must implement `\Drupal\commerce_order\OrderProcessorInterface`.

The following is an example for a module's `*.services.yml` file.

```
  # Order refresh process to apply taxes.
  # We set the priority very low so it calculates last.
  commerce_demo.order_process.taxes:
    class: Drupal\commerce_demo\OrderProcessor\ApplyTaxAdjustments
    tags:
      - { name: commerce_order.order_processor, priority: -300 }
```
