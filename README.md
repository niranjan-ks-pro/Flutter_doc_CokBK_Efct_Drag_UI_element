# Flutter_doc_CokBK_Efct_Drag_UI_element
 https://docs.flutter.dev/cookbook/effects/drag-a-widget#press-and-drag
Drag a UI element
=================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Drag a UI element](https://docs.flutter.dev/cookbook/effects/drag-a-widget)

Drag and drop is a common mobile app interaction. As the user long presses (sometimes called *touch & hold*) on a widget, another widget appears beneath the user's finger, and the user drags the widget to a final location and releases it. In this recipe, you'll build a drag-and-drop interaction where the user long presses on a choice of food, and then drags that food to the picture of the customer who is paying for it.

The following animation shows the app's behavior:

![Ordering the food by dragging it to the person](https://docs.flutter.dev/assets/images/docs/cookbook/effects/DragAUIElement.gif)

This recipe begins with a prebuilt list of menu items and a row of customers. The first step is to recognize a long press and display a draggable photo of a menu item.

[](https://docs.flutter.dev/cookbook/effects/drag-a-widget#press-and-drag)Press and drag
----------------------------------------------------------------------------------------

Flutter provides a widget called [`LongPressDraggable`](https://api.flutter.dev/flutter/widgets/LongPressDraggable-class.html) that provides the exact behavior that you need to begin a drag-and-drop interaction. A `LongPressDraggable` widget recognizes when a long press occurs and then displays a new widget near the user's finger. As the user drags, the widget follows the user's finger. `LongPressDraggable` gives you full control over the widget that the user drags.

Each menu list item is displayed with a custom `MenuListItem` widget.

content_copy

```
MenuListItem(
  name: item.name,
  price: item.formattedTotalItemPrice,
  photoProvider: item.imageProvider,
)
```

Wrap the `MenuListItem` widget with a `LongPressDraggable` widget.

content_copy

```
LongPressDraggable<Item>(
  data: item,
  dragAnchorStrategy: pointerDragAnchorStrategy,
  feedback: DraggingListItem(
    dragKey: _draggableKey,
    photoProvider: item.imageProvider,
  ),
  child: MenuListItem(
    name: item.name,
    price: item.formattedTotalItemPrice,
    photoProvider: item.imageProvider,
  ),
);
```

In this case, when the user long presses on the `MenuListItem` widget, the `LongPressDraggable` widget displays a `DraggingListItem`. This `DraggingListItem` displays a photo of the selected food item, centered beneath the user's finger.

The `dragAnchorStrategy` property is set to [`pointerDragAnchorStrategy`](https://api.flutter.dev/flutter/widgets/pointerDragAnchorStrategy.html). This property value instructs `LongPressDraggable` to base the `DraggableListItem`'s position on the user's finger. As the user moves a finger, the `DraggableListItem` moves with it.

Dragging and dropping is of little use if no information is transmitted when the item is dropped. For this reason, `LongPressDraggable` takes a `data` parameter. In this case, the type of `data` is `Item`, which holds information about the food menu item that the user pressed on.

The `data` associated with a `LongPressDraggable` is sent to a special widget called `DragTarget`, where the user releases the drag gesture. You'll implement the drop behavior next.

[](https://docs.flutter.dev/cookbook/effects/drag-a-widget#drop-the-draggable)Drop the draggable
------------------------------------------------------------------------------------------------

The user can drop a `LongPressDraggable` wherever they choose, but dropping the draggable has no effect unless it's dropped on top of a `DragTarget`. When the user drops a draggable on top of a `DragTarget` widget, the `DragTarget` widget can either accept or reject the data from the draggable.

In this recipe, the user should drop a menu item on a `CustomerCart` widget to add the menu item to the user's cart.

content_copy

```
CustomerCart(
  hasItems: customer.items.isNotEmpty,
  highlighted: candidateItems.isNotEmpty,
  customer: customer,
);
```

Wrap the `CustomerCart` widget with a `DragTarget` widget.

content_copy

```
DragTarget<Item>(
  builder: (context, candidateItems, rejectedItems) {
    return CustomerCart(
      hasItems: customer.items.isNotEmpty,
      highlighted: candidateItems.isNotEmpty,
      customer: customer,
    );
  },
  onAccept: (item) {
    _itemDroppedOnCustomerCart(
      item: item,
      customer: customer,
    );
  },
)
```

The `DragTarget` displays your existing widget and also coordinates with `LongPressDraggable` to recognize when the user drags a draggable on top of the `DragTarget`. The `DragTarget` also recognizes when the user drops a draggable on top of the `DragTarget` widget.

When the user drags a draggable on the `DragTarget` widget, `candidateItems` contains the data items that the user is dragging. This draggable allows you to change what your widget looks like when the user is dragging over it. In this case, the `Customer` widget turns red whenever any items are dragged above the `DragTarget` widget. The red visual appearance is configured with the `highlighted` property within the `CustomerCart` widget.

When the user drops a draggable on the `DragTarget` widget, the `onAccept` callback is invoked. This is when you get to decide whether or not to accept the data that was dropped. In this case, the item is always accepted and processed. You might choose to inspect the incoming item to make a different decision.

Notice that the type of item dropped on `DragTarget` must match the type of the item dragged from `LongPressDraggable`. If the types are not compatible, then the `onAccept` method isn't invoked.

With a `DragTarget` widget configured to accept your desired data, you can now transmit data from one part of your UI to another by dragging and dropping.

In the next step, you update the customer's cart with the dropped menu item.

[](https://docs.flutter.dev/cookbook/effects/drag-a-widget#add-a-menu-item-to-a-cart)Add a menu item to a cart
--------------------------------------------------------------------------------------------------------------

Each customer is represented by a `Customer` object, which maintains a cart of items and a price total.

content_copy

```
class Customer {
  Customer({
    required this.name,
    required this.imageProvider,
    List<Item>? items,
  }) : items = items ?? [];

  final String name;
  final ImageProvider imageProvider;
  final List<Item> items;

  String get formattedTotalItemPrice {
    final totalPriceCents =
        items.fold<int>(0, (prev, item) => prev + item.totalPriceCents);
    return '\$${(totalPriceCents / 100.0).toStringAsFixed(2)}';
  }
}
```

The `CustomerCart` widget displays the customer's photo, name, total, and item count based on a `Customer` instance.

To update a customer's cart when a menu item is dropped, add the dropped item to the associated `Customer` object.

content_copy

```
void _itemDroppedOnCustomerCart({
  required Item item,
  required Customer customer,
}) {
  setState(() {
    customer.items.add(item);
  });
}
```
