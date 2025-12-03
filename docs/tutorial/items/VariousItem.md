# 更多样的物品

> 让我们继续深入

## 设置物品属性

我们在上一份文档中创建了一个最简单的自定义物品：`custom_item`。这个物品不属于工具，不能食用，不能饮用，手持它右键也不会有任何事发生，这很显然是一个没有任何用处的物品，并不是我们所想要的。这是因为在**ModItems**中注册这个物品时，我们没有为它设置任何属性。

这是我们在上一个文档中的物品注册代码：

```
public class ModItems {

    ...

    public static final Supplier<Item> CUSTOM_ITEM = ITEMS.registerSimpleItem("custom_item", new Item.Properties());

    ...

}
```

聪明的你应该注意到了，在这段注册代码的最后，有一个称为`new Item.Properties()`的参数，这个参数正是本节文档的重点：为`new Item.Properties()`添加更多设置。

### 简单的食物物品属性设置

让我们从最简单的食物物品开始，创建一个可食用的物品：

```
public class ModItems {

    ...

    public static final Supplier<Item> CUSTOM_ITEM = ITEMS.registerSimpleItem("custom_item", new Item.Properties()
            .food(new FoodProperties.Builder()
                    .nutrition(3)
                    .saturationModifier(0.3f)
                    .alwaysEdible()
                    .build()
            )
            .stacksTo(16)
    );

    ...

}
```

注意这段注册代码与我们最初的物品注册代码的不同之处，在`new Item.Properties()`参数之后，多出了一项`.food()`方法调用，它其实是一类数据组件(Data Component)，我们会在之后的文档中学习并使用它。但是就目前来说，你只需要掌握如何使用它的方法即可。

在`.food()`方法参数中，我们创建了一个新的食物物品属性构造器：`new FoodProperties.Builder()`，然后进行了以下几个参数的设置：

- **.nutrition()**：这个参数设置了食用该食物后玩家能恢复的血量。
- **saturationModifier()**：这个参数设置了食用该食物后玩家可恢复的饱和度。
- **.alwaysEdible()**：这个参数决定了该食物是否在玩家饱和度已满的状态下仍然可食用。

当然还有我们之前提到过的：**.stacksTo()**，它可以设置物品的最大可堆叠数量。如果你不设置这一项，最大堆叠数量就会是默认的64个

通过这几个属性的设置，你已经成功构建了一个可以食用的物品。此时进入游戏，手持你的自定义食物长按右键就能吃下它了！

## 更复杂的食物物品

简简单单的食物实在是太无聊了，我们需要更多样的食物，我们需要为mod添加饮品，或者我们想要玩家食用物品后获得各种奇奇怪怪的buff，这些想法都应该如何实现呢？

为了优雅的实现以上的两个想法，我们都需要创建自定义物品类。

### 可饮用的物品

首先让我们开始着手实现一个可以饮用的物品，我们的需求是：通过这个自定义物品类注册的饮品可以**在食用时显示饮用的动画和音效，自由设置它的饮用时间以及饮用后返回的物品。**

请在**java/com/sanjin/tutorial**路径下，也就是和**register**软件包的同级目录下新建一个软件包，命名为**item**。在**item**目录下，我们需要新建一个java文件，命名为**DrinkableItem**。

打开**DrinkableItem.java**文件，我们需要它继承**Item**类，像这样编写：

```
public class DrinkableItem extends Item {

}
```

此时你会发现你的文件出现了红色波浪线，也就是报错。不用紧张，让我们先理解现在正在干什么：你可以将这样的继承关系简单理解为父子关系，**Item**类是父类，我们自己创建的**DrinkableItem**是子类。子类想要继承父类就必须要实现父类的抽象方法，并且要有自己的构造函数，这也是为什么我们的文件现在出现了报错的原因。

另外不要导入错误的Item类！Minecraft源码中存在两个**Item**类，这里需要导入的是**import net.minecraft.world.item.Item;**路径下的**Item**类。

现在让我们来完善代码：

```
public class DrinkableItem extends Item {

    public DrinkableItem(Properties properties) {
        super(properties);
    }

}
```

父类**Item**中并没有抽象方法要求我们实现，所以写上构造函数这样就已经完成了子类**DrinkableItem**的构建。在构造函数中，`super()`方法是为了调用所有父类中的方法，让子类**DrinkableItem**也能具备所有父类的功能。

当然，我们还没有实现我们想要的功能，我们的目的是能够食用时显示饮用的动画和音效，自由设置它的饮用时间以及饮用后返回的物品。所以我们要这样为子类添加参数：`ItemStack remainderItem`。

```
public class DrinkableItem extends Item{

    private final ItemStack remainderItem;

    public DrinkableItem(Properties properties, ItemStack remainderItem) {
        super(properties);

        this.remainderItem = remainderItem;
    }

}
```

添加了这个参数之后，主类构造函数中也必须要有`ItemStack remainderItem`，要求在我们在调用DrinkableItem构建饮品时必须要传入一个ItemStack类型参数。

光传入一个参数还不够，饮用后返回物品的真正逻辑还没有实现，这里我们要重写父类的一个方法：

```
public class DrinkableItem extends Item{

    private final ItemStack remainderItem;

    public DrinkableItem(Properties properties, ItemStack remainderItem) {
        super(properties);

        this.remainderItem = remainderItem;
    }

    @Override
    public @NotNull ItemStack finishUsingItem(@NotNull ItemStack stack, @NotNull Level level, @NotNull LivingEntity entity) {

        ItemStack result = super.finishUsingItem(stack, level, entity);

        if (entity instanceof Player player && !remainderItem.isEmpty()) {
            ItemStack remainder = remainderItem.copy();
            if (!player.getInventory().add(remainder)) {
                player.drop(remainder, false);
            }
        }

        return result;
    }

}
```

`finishUsingItem()`方法来源于父类，子类可以通过重写父类的方法来实现自己的功能，但是方法名以及方法的参数都不能更改。如果不重写，就会在`super()`函数调用时默认执行父类的方法。

根据Java文件的规范，重写父类方法时需要在方法前添加一个`@Override`标明这是一个重写的方法。当然如果不标也不会报错。

我简单解释一下这个方法中的逻辑内容。这个方法会在生物持有该物品时进行检测，当该物品被使用后，系统给予该生物一个特定的物品。对应方法中的参数就是`LivingEntity entity`和`ItemStack stack`。而我们在方法逻辑中将DrinkableItem中的私有参数`private final ItemStack remainderItem`赋给方法中的`ItemStack stack`。

现在返回物品的逻辑已经实现了，让我们实现下一个需求，食用时显示饮用的动画和音效，自定义饮用的时间。这里我们需要再次借用数据组件的力量：

```
public class DrinkableItem extends Item {

    private final ItemStack remainderItem;

    public DrinkableItem(@NotNull Properties properties, float useDuration, ItemStack remainderItem) {

        super(properties.component(DataComponents.CONSUMABLE, Consumabl.builder()
                .consumeSeconds(useDuration)
                .animation(ItemUseAnimation.DRINK)
                .sound(SoundEvents.GENERIC_DRINK)
                .soundAfterConsume(SoundEvents.GENERIC_DRINK)
                .hasConsumeParticles(false)
                .build())
            );

        this.remainderItem = remainderItem;
    }

    @Override
    public @NotNull ItemStack finishUsingItem(@NotNull ItemStack stack, @NotNull Level level, @NotNull LivingEntity entity) {

        ItemStack result = super.finishUsingItem(stack, level, entity);

        if (entity instanceof Player player && !remainderItem.isEmpty()) {
            ItemStack remainder = remainderItem.copy();
            if (!player.getInventory().add(remainder)) {
                player.drop(remainder, false);
            }
        }

        return result;
    }
}
```

你会发现`super()`函数中的参数`properties`后面多了很长的一串内容。`properties`实际上就是我们在注册物品时要处理的属性设置，在这里，我们提前为未来将要调用**DrinkableItem**类来注册的物品的属性加上了一个`.component()`，也就是数据组件。

在`.component()`方法的参数中，我们又调用了一个构造器`Consumabl.builder()`，这是一个“持续性”数据组件构造器，下面是它的几项参数设置：

- **consumeSeconds()**：这项属性正是我们要自定义的**饮用的时间**，我们将**DrinkableItem**函数构造器中要传入的参数`float useDuration`传入这项属性设置中，就可以完成对**饮用的时间**的自定义。
- **.animation()**：这项属性决定了该物品在被使用时调用的是什么动画，这里我们需要的是饮用动画，所以传入`ItemUseAnimation.DRINK`。实际上`ItemUseAnimation`还包含了许多动画，你可以自由选择你喜欢的，比如拉弓的动画，格挡的动画甚至使用望远镜的动画。（~~当然在食用物品时使用这些动画看起来可能会有些诡异~~）
- **.sound()**：这项属性设置了饮用时调用的音效，示例中使用的是饮水/药水的音效，当然你也可以选择其他的音效。
- **.soundAfterConsume()**：这项属性设置了饮用结束后调用的音效，没错，饮用结束的音效和饮用时的音效是分开来设置的。
- **.hasConsumeParticles()**：这项属性设置了在饮用时是否会出现粒子效果，就像食用食物时会出现食物的碎片离子效果一样。当然饮用时一般是不会出现这样的粒子效果的，所以这里我们选择false（如果你想要粒子效果，选择true即可）。

可见数据组件的功能是十分强大的，可以自由的为物品添加自定义的效果,我们后面会更系统的学习它，这里暂且不展开讲。现在我们已经完成了对**DrinkableItem**类的构造。现在让我们回到**ModItems.java**文件，调用**DrinkableItem**类来注册一个新的饮品`ENERGY_DRINK`：

```
public class ModItems{

    ...

    public static final Supplier<Item> ENERGY_DRINK = ITEMS.registerItem("energy_drink",
    properties -> new DrinkableItem(
                    properties.food(new FoodProperties.Builder()
                                    .nutrition(3)
                                    .saturationModifier(0.3f)
                                    .alwaysEdible()
                                    .build()
                            )
                            .stacksTo(16),
                    1.2f,
                    new ItemStack(Items.GLASS_BOTTLE)
            )
    );

    ...

}
```

`ENERGY_DRINK`的注册与我们之前注册的普通食物物品的最大区别就是换用了一个新的物品类：**DrinkableItem**。并且我们传入了构造**DrinkableItem**所需要的所有属性：**properties**，**1.2f**（对应`float useDuration`），**new ItemStack(Items.GLASS_BOTTLE)**（对应`ItemStack remainderItem`）。其中对于**properties**属性我们还进行了两个额外的设置：`.food()`和`.stacksTo()`，从而设置了改饮品的食物属性和最大堆叠数。当然，这些都是可以自由选择和调整的，并不是物品注册的必要条件。

**1.2f**是原版Minecraft中通用的饮用时间，你也可以根据自己的需要调整。**new ItemStack(Items.GLASS_BOTTLE)**这个方法中传入的参数：`Items.GLASS_BOTTLE`可以替换为任何已注册的物品，包括原版和你的自定义物品。

### 带有Buff的食物

可自定义的饮品还不够酷？让我们试试为食物添加Buff！在原版中，部分食物例如腐肉，金苹果在被食用后会给玩家增添Buff，现在我们的自定义食物也想要实现这个效果，改如何着手呢？

从**Minecraft 1.21.3 + NeoForge 21.3.94**开始，食物效果不再通过`FoodProperties.Builder#effect()`添加，而是通过**数据组件Consumable + ConsumeEffect**来完成。我们需要这样实现：

```
public class ModItems {

    ...

    public static final Supplier<Item> GOLDEN_BERRY = ITEMS.registerSimpleItem("golden_berry",
            new Item.Properties()
                    .food(
                            new FoodProperties(4, 0.6f, true), // 营养值，饱和度，是否总能食用
                            Consumable.builder()
                                    .consumeSeconds(1.6f) // 食用时间（秒），可按需修改
                                    .onConsume(new ApplyStatusEffectsConsumeEffect(
                                            new MobEffectInstance(MobEffects.REGENERATION, 200, 0), 1.0f))
                                    .onConsume(new ApplyStatusEffectsConsumeEffect(
                                            new MobEffectInstance(MobEffects.HUNGER, 100, 0), 0.25f))
                                    .build()
                    )
                    .stacksTo(16)
    );

    ...

}
```

上面的示例为`golden_berry`添加了两个效果：
- `MobEffects.REGENERATION`：持续200 tick（也就是10秒）的生命恢复，100%触发。
- `MobEffects.HUNGER`：持续100 tick（5秒）的饥饿效果，25%概率触发。

在1.21.3中，`.food(FoodProperties, Consumable)`会同时挂上“食物数值”和“食用行为”两个数据组件；`Consumable#onConsume()`负责添加具体的效果逻辑。你可以连续调用多次`.onConsume()`来堆叠不同的正面/负面效果，并为每个效果单独设置触发概率。记得保持必要的导入：`FoodProperties`（`net.minecraft.world.food`）、`Consumable`与`ApplyStatusEffectsConsumeEffect`（`net.minecraft.world.item.component`与`net.minecraft.world.item.consume_effects`），以及`MobEffectInstance`。

注册好后，别忘了像之前那样在语言文件中为`item.tutorial.golden_berry`添加中英文名称，以及在`textures/item`、`models/item`中准备好贴图和模型文件。

## 更多好用的Item属性

在实际开发中，除了食物和饮品，我们还会频繁用到一些常见的属性配置：

```
public class ModItems {

    ...

    public static final Supplier<Item> SPECIAL_CORE = ITEMS.registerSimpleItem("special_core",
            new Item.Properties()
                    .rarity(Rarity.UNCOMMON) // 稀有度：COMMON/UNCOMMON/RARE/EPIC
                    .fireResistant() // 掉进火焰/岩浆不被烧毁
                    .durability(512) // 耐久度：会显示耐久条
                    .setNoRepair() // 禁止铁砧修复（可选）
                    .craftRemainder(Items.BLAZE_ROD) // 合成后返回的物品（可选）
    );

    ...

}
```


## 让物品“动”起来：交互、动画与冷却

如果你希望物品在右键时触发特殊行为（例如蓄力、冷却、播放动画），可以选择创建一个新的物品类并重写`use()`等方法。下面是一个简单示例：

```
public class BlinkStoneItem extends Item {

    public BlinkStoneItem(Properties properties) {
        super(properties);
    }

    @Override
    public int getUseDuration(@NotNull ItemStack stack) {
        return 24; // 1.2秒的蓄力时间（20 tick = 1秒）
    }

    @Override
    public @NotNull UseAnim getUseAnimation(@NotNull ItemStack stack) {
        return UseAnim.BOW; // 使用拉弓的动画
    }

    @Override
    public @NotNull InteractionResultHolder<ItemStack> use(@NotNull Level level, @NotNull Player player, @NotNull InteractionHand hand) {
        ItemStack stack = player.getItemInHand(hand);

        if (!level.isClientSide()) {
            player.getCooldowns().addCooldown(this, 40); // 2秒冷却
            player.teleportTo(player.getX(), player.getY() + 1.5, player.getZ()); // 向上瞬移一点点
        }

        return InteractionResultHolder.sidedSuccess(stack, level.isClientSide());
    }
}
```

要使用它，只需要像注册其他物品一样在**ModItems**中注册`BlinkStoneItem`即可。几点注意：
- 只在`!level.isClientSide()`时处理实际逻辑，避免客户端/服务端重复执行。
- 冷却通过`player.getCooldowns().addCooldown()`实现，单位是tick。
- `getUseDuration()`与`getUseAnimation()`控制蓄力时间与动画表现。

## 友好的物品描述（工具提示）

如果你想在物品上添加描述文字或提示效果，可以重写`appendHoverText()`方法，结合语言文件的翻译键来实现：

```
public class EnergyDrinkItem extends DrinkableItem {

    public EnergyDrinkItem(@NotNull Properties properties, float useDuration, ItemStack remainderItem) {
        super(properties, useDuration, remainderItem);
    }

    @Override
    public void appendHoverText(@NotNull ItemStack stack, Item.@NotNull TooltipContext context, @NotNull List<Component> tooltipComponents, @NotNull TooltipFlag tooltipFlag) {
        tooltipComponents.add(Component.translatable("tooltip.tutorial.energy_drink").withStyle(ChatFormatting.AQUA));
        tooltipComponents.add(Component.literal("恢复并提神！").withStyle(ChatFormatting.GRAY));
    }
}
```

在语言文件中添加：

```
{
    "item.tutorial.energy_drink": "Energy Drink",
    "tooltip.tutorial.energy_drink": "Gives you a short burst of speed!"
}
```

这样，鼠标悬停时就能显示额外的提示文本。你也可以根据`tooltipFlag.isAdvanced()`添加更详细的调试信息。
