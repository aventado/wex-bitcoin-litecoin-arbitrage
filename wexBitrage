"""

In 2018 this script might make a few cents to a few dollars per day but generally
will not be worth it to run, feel free to fork and use as a base for something else

Known Issues:
- Does not calculate max trade size based on each leg of the trade (must be in and out of each position seamlessly)
- Does not calculate alternative exit strategies (if caught in leg and selling is unprofitable because of a lag in execution, can we move to another pair and get back to USD?)
- Probably more 

Donate BTC: 3EYfd42eD5dZCWVBXYzrF5wMMR4ERLs68b
Donate ETH: 0x020FB5A3DfcA8a23c2D3a2d8fe37E277ce183011
Donate LTC: MD1kYZZALcZRRidE4E4ey4LhnT5C5eEfWU (Segwit)

"""

import httplib
import urllib
import json
import hashlib
import hmac
import time


class api:
 __api_key	= '';
 __api_secret	= '';
 __nonce_v	= 1;
 __wait_for_nonce = False

 def __init__(self,api_key,api_secret,wait_for_nonce=False):
  self.__api_key = api_key
  self.__api_secret = api_secret
  self.__wait_for_nonce = wait_for_nonce

 def __nonce(self):
   if self.__wait_for_nonce: time.sleep(1)
   self.__nonce_v = str(time.time()).split('.')[0]

 def __signature(self, params):
  return hmac.new(self.__api_secret, params, digestmod=hashlib.sha512).hexdigest()

 def __api_call(self,method,params):
  self.__nonce()
  params['method'] = method
  params['nonce'] = str(self.__nonce_v)
  params = urllib.urlencode(params)
  headers = {"Content-type" : "application/x-www-form-urlencoded",
                      "Key" : self.__api_key,
		     "Sign" : self.__signature(params)}
  conn = httplib.HTTPSConnection("wex.nz")
  conn.request("POST", "/tapi", params, headers)
  response = conn.getresponse()
  data = json.load(response)
  conn.close()
  return data
  
 def get_param(self, couple, param):
  conn = httplib.HTTPSConnection("wex.nz")
  conn.request("GET", "/api/2/"+couple+"/"+param)
  response = conn.getresponse()
  data = json.load(response)
  conn.close()
  return data
 
 def getInfo(self):
  return self.__api_call('getInfo', {})

 def TransHistory(self, tfrom, tcount, tfrom_id, tend_id, torder, tsince, tend):
  params = {
   "from"	: tfrom,
   "count"	: tcount,
   "from_id"	: tfrom_id,
   "end_id"	: tend_id,
   "order"	: torder,
   "since"	: tsince,
   "end"	: tend}
  return self.__api_call('TransHistory', params)
 
 def TradeHistory(self, tfrom, tcount, tfrom_id, tend_id, torder, tsince, tend, tpair):
  params = {
   "from"	: tfrom,
   "count"	: tcount,
   "from_id"	: tfrom_id,
   "end_id"	: tend_id,
   "order"	: torder,
   "since"	: tsince,
   "end"	: tend,
   "pair"	: tpair}
  return self.__api_call('TradeHistory', params)

 def ActiveOrders(self, tpair):
  params = { "pair" : tpair }
  return self.__api_call('ActiveOrders', params)

 def Trade(self, tpair, ttype, trate, tamount):
  params = {
   "pair"	: tpair,
   "type"	: ttype,
   "rate"	: trate,
   "amount"	: tamount}
  return self.__api_call('Trade', params)
  
 def CancelOrder(self, torder_id):
  params = { "order_id" : torder_id }
  return self.__api_call('CancelOrder', params)


# INSERY OUR API + SECRET
infom = api('key', 'secret',False)


# TRADE LOGIC AND LOOP EXECUTION
url = "https://wex.nz/api/3/ticker/btc_usd-ltc_btc-ltc_usd"
bot_capital = 15 # CAPITAL (USD) TO EXECUTE TRADES WITH -- KEEP SIZE SMALL
gain = 0
attempts = 0


# RUN CONTINUOUSLY
while True:
    try:
        response = urllib.urlopen(url)
        data = json.load(response)
        print "------"
    except URLError, e:
        print "No response from server"

    first_leg = (bot_capital/data['btc_usd']['buy'])*.998
    second_leg = (first_leg/data['ltc_btc']['sell'])*.998
    final_leg = (second_leg*data['ltc_usd']['sell'])*.998
    
    if final_leg>bot_capital:
        infom.Trade('btc_usd','buy',data['btc_usd']['buy'], .001)
        if infom.getInfo()['return']['funds']['btc']>0:
            infom.Trade('ltc_btc', 'buy', data['ltc_btc']['buy'], (.001/data['ltc_btc']['buy']))
        if infom.getInfo()['return']['funds']['ltc']>0:
            infom.Trade('ltc_usd', 'sell', data['ltc_usd']['sell'], infom.getInfo()['return']['funds']['ltc'])
        print "SUCCESSFUL TRADE!"
        attempts += 1
        print infom.getInfo()['return']['funds']['usd']
                    
    else:
        print "Trade not profitable."
        attempts += 1
        print attempts
