## QStyledItemDelegate 和  QItemDelegate的区别

**QStyledItemDelegate** 和 **QItemDelegate** 都是继承 **QAbstractItemDelegate**，官方文档上说这两个类基本一样， 唯一不同是 **QStyledItemDelegate**  可以被 **StyleSheet** 影响。

![image-01](https://github.com/mingxingren/Notes/raw/master/resource/photo/image-2021050801.png)



接下来查看 **QStyledItemDelegate** 和 **QItemDelegate** 的 **paint**() 源码: 

```c++
// QStyledItemDelegate 使用 QStyle 的 drawControl 方法绘制可受到参数 widget 的样式影响
void QStyledItemDelegate::paint(QPainter *painter,
        const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    Q_ASSERT(index.isValid());

    QStyleOptionViewItem opt = option;
    initStyleOption(&opt, index);

    const QWidget *widget = QStyledItemDelegatePrivate::widget(option);
    QStyle *style = widget ? widget->style() : QApplication::style();
    style->drawControl(QStyle::CE_ItemViewItem, &opt, painter, widget);
}
```

```c++
// QItemDelegate 使用 QPainter 原生绘制图形, Qss无法影响到 QItemDelegate

void QItemDelegate::paint(QPainter *painter,
                          const QStyleOptionViewItem &option,
                          const QModelIndex &index) const
{
    Q_D(const QItemDelegate);
    Q_ASSERT(index.isValid());

    QStyleOptionViewItem opt = setOptions(index, option);

    // prepare
    painter->save();
    if (d->clipPainting)
        painter->setClipRect(opt.rect);

    // get the data and the rectangles

    QVariant value;

    QPixmap pixmap;
    QRect decorationRect;
    value = index.data(Qt::DecorationRole);
    if (value.isValid()) {
        // ### we need the pixmap to call the virtual function
        pixmap = decoration(opt, value);
        if (value.userType() == QMetaType::QIcon) {
            d->tmp.icon = qvariant_cast<QIcon>(value);
            d->tmp.mode = d->iconMode(option.state);
            d->tmp.state = d->iconState(option.state);
            const QSize size = d->tmp.icon.actualSize(option.decorationSize,
                                                      d->tmp.mode, d->tmp.state);
            decorationRect = QRect(QPoint(0, 0), size);
        } else {
            d->tmp.icon = QIcon();
            decorationRect = QRect(QPoint(0, 0), pixmap.size());
        }
    } else {
        d->tmp.icon = QIcon();
        decorationRect = QRect();
    }

    QRect checkRect;
    Qt::CheckState checkState = Qt::Unchecked;
    value = index.data(Qt::CheckStateRole);
    if (value.isValid()) {
        checkState = static_cast<Qt::CheckState>(value.toInt());
        checkRect = doCheck(opt, opt.rect, value);
    }

    QString text;
    QRect displayRect;
    value = index.data(Qt::DisplayRole);
    if (value.isValid() && !value.isNull()) {
        text = d->valueToText(value, opt);
        displayRect = d->displayRect(index, opt, decorationRect, checkRect);
    }

    // do the layout

    doLayout(opt, &checkRect, &decorationRect, &displayRect, false);

    // draw the item

    drawBackground(painter, opt, index);
    drawCheck(painter, opt, checkRect, checkState);
    drawDecoration(painter, opt, decorationRect, pixmap);
    drawDisplay(painter, opt, displayRect, text);
    drawFocus(painter, opt, displayRect);

    // done
    painter->restore();
}

void QItemDelegate::drawBackground(QPainter *painter,
                                   const QStyleOptionViewItem &option,
                                   const QModelIndex &index) const
{
    if (option.showDecorationSelected && (option.state & QStyle::State_Selected)) {
        QPalette::ColorGroup cg = option.state & QStyle::State_Enabled
                                  ? QPalette::Normal : QPalette::Disabled;
        if (cg == QPalette::Normal && !(option.state & QStyle::State_Active))
            cg = QPalette::Inactive;

        painter->fillRect(option.rect, option.palette.brush(cg, QPalette::Highlight));
    } else {
        QVariant value = index.data(Qt::BackgroundRole);
        if (value.canConvert<QBrush>()) {
            QPointF oldBO = painter->brushOrigin();
            painter->setBrushOrigin(option.rect.topLeft());
            painter->fillRect(option.rect, qvariant_cast<QBrush>(value));
            painter->setBrushOrigin(oldBO);
        }
    }
}
```

