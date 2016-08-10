from quantopian.pipeline import Pipeline
from quantopian.algorithm import attach_pipeline, pipeline_output
from quantopian.pipeline.factors import CustomFactor
from quantopian.pipeline.data.quandl import cboe_vix, cboe_vxv
import pandas as pd
import numpy as np

def initialize(context):
    schedule_function(func=rebal, date_rule=date_rules.month_end(days_offset=3), time_rule=time_rules.market_open(hours=2))
    pipe = Pipeline()
    attach_pipeline(pipe, 'vix_pipeline')  
    v = Get_VIXVXV()    
    pipe.add(v, 'vixvxv')

class Get_VIXVXV(CustomFactor): 
    # close vs vix_close.  always a pleasure.
    inputs = [cboe_vix.vix_close, cboe_vxv.close]
    window_length = 1    
    def compute(self, today, assets, out, vix, vxv):
        out[:] = (vix[-1]/vxv[-1]) * np.ones(len(assets))      

def before_trading_start(context, data):
    output = pipeline_output('vix_pipeline')
    output = output.dropna()    
    context.vixvxv = output.loc[symbol('SPY')]['vixvxv']
    pass
    
def handle_data(context,data):
    pass

def rebal(context,data):
    risk_on = context.vixvxv < 0.9
      
    ronn_asset = sid(19662)
    roff_asset = sid(19659)
    hedge_asset = sid(8554)    
        
    if (risk_on):
        longAsset = ronn_asset
        longWeight = 1.0
        exit = roff_asset
    else:
        longAsset = roff_asset
        longWeight = 1.7
        exit = ronn_asset
    
    order_target_percent(longAsset, longWeight)
    order_target_percent(exit, 0.0)
    order_target_percent(hedge_asset, -1.0)
