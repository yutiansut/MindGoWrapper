# MindGoWrapper

Some wrapper for sh*t quant platform MindGo.

## Usage

Reset kernel every time before running. (`pwd` changes every time backtest is run.)

### Cell 0

Assume your Jupyter notebook is in the root folder:

```shell
!git clone https://github.com/Jamesits/MindGoWrapper.git
!sh -c "cd MindGoWrapper; git pull"
```

Non-root `pwd` is more complicated; you may not be able to use it in backtest panel. 

### Cell 1

```
%load_ext backtest
```

### Cell 2

The actual backtest code：

```python
%%backtest --start 2015-9-1 --end 2015-10-1 --capital-base 100000 --data-frequency minute --output -
# data-frequency: days, minute or tick

from MindGoWrapper.wrapper import Wrapper
from MindGoWrapper.map import Map
from MindGoWrapper.scheduler import Scheduler
import logging

logging.basicConfig(level=logging.INFO)
w = Wrapper()

config = Map({
    # 选股数量
    'security_count': 10,
    # 资金用于投资的比例
    'currency_use_percent': 0.8,
    # 购买
    'purchase': Map({
        # 刚买入时建仓百分比
        'initial_purchase': 0.05,
    }),
})

# callback_function will be executed twice every 22 days, at day 0 and 10
w.scheduler.schedule(callback_function, Scheduler.timeslot([0, 10], 22, Scheduler.Unit.DAY, Scheduler.Slot.BEFORE))

# initialize MindGo platform
w.takeown(globals(), config)
```

## API Doc

```python
from MindGoWrapper.wrapper import Wrapper
w = Wrapper()

# initialize platform, execute anywhere in the backtest code
w.takeown() 

# get stock information
w.get_current_price(symbol)
w.is_paused(symbol)

# prepare to buy
w.create_portfolio(symbol)

# prepare to sell out
w.remove_portfolio(symbol)

# get a Portfolio object
w.get_portfolio_detail(symbol)

# set purchase goal
w.update_portfolio_object_value(symbol, goal_value)

# batch prepare
# buy in everything in the list
# sell out everything not in the list
w.set_portfolios([symbol_list])

from MindGoWrapper.map import Map

# an object whose attributes can be accessed in both dict style and dot notation
m = Map({dict}, key=value)
m['key']
m.key

from MindGoWrapper.scheduler import Scheduler

# run at 1st and 11th day every 22 days
# Unit: DAY or TICK (tick is the minimum frequency of backtest)
# Slot: BEFORE or AFTER all transaction happens
w.scheduler.schedule(callback, Scheduler.timeslot([0, 10], 22, Scheduler.Unit.DAY, Scheduler.Slot.BEFORE))

# Other functions

# Iterate through a pandas.Dataframe df by ascending/descending sequence of seq
MindGoWrapper.utils.df_iter(df, seq, rank=False, ascending=True)
``` 