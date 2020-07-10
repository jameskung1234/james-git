# james-git
james git
"""
#Akkarawat Mansap

#Credit1 : Nattapon Soomtha (กองทุนความมั่งคั่งแห่งชาติ Training)
#Credit2 : b2 spetsnaz club
#Credit3 : TEERACHAI RATTANABUNDITSAKUL

#----------------------------------------------------------------

# 1.Install library ที่เราจำเป็นจะต้องใช้ในการส่งคำสั่งและการตรวจเช็คออเดอร์
# - ccxt library ที่เป็นที่นิยมในการเชื่อม API กับ Exchange ต่างๆได้ง่ายขึ้นมาก
# - pandas library จะช่วยในการแปลงข้อมูลที่เราดึงมาจาก Exchange แปลงให้
#   เป็นตารางเพื่อให้ง่ายต่อการตรวจสอบออเดอร์
# - json จะใช้สำหรับการดึงข้อมูลที่เราต้องการมาจาก Exchange

"""

pip install ccxt
pip install pandas



# Config Session

import ccxt
import json
import pandas as pd
import time

# api and secret

apiKey = ''         #input("Enter API Key:")      
secret = ''         #input("Enter Secret Key:")
subaccount = ''                                      #input("Enter Sub Account Name or 0 for Main:")


# Exchange Detail
exchange = ccxt.ftx({
    'apiKey' : apiKey ,'secret' : secret ,'enableRateLimit': True
})

# Sub Account Check

if subaccount == "0":
  print("This is Main Account")
else:
  exchange.headers = {
   'FTX-SUBACCOUNT': subaccount,
  }

# Global Variable Setting
pair = 'BTC-PERP'
tf = '5m'
buyRecord = []
minOrder = min(buyRecord, default=0.0)
maxOrder = max(buyRecord, default=100000000.0)

# Get Historical Price Data

def priceHistdata():
  
  try:
    priceData = pd.DataFrame(exchange.fetch_ohlcv(pair,tf))
  except ccxt.NetworkError as e:
    print(exchange.id, 'fetch_ohlcv failed due to a network error:', str(e))
    priceData = pd.DataFrame(exchange.fetch_ohlcv(pair,tf))
  except ccxt.ExchangeError as e:  
    print(exchange.id, 'fetch_ohlcv failed due to exchange error:', str(e))
    priceData = pd.DataFrame(exchange.fetch_ohlcv(pair,tf))
  except Exception as e:
    print(exchange.id, 'fetch_ohlcv failed with:', str(e))
    priceData = pd.DataFrame(exchange.fetch_ohlcv(pair,tf))

  return priceData

#Get Last Price

def getPrice():

  try:
    r1 = json.dumps(exchange.fetch_ticker(pair))
    dataPrice = json.loads(r1)
    #print(exchange)
    #print(pair + '=',dataPrice['last'])
  except ccxt.NetworkError as e:
    r1 = json.dumps(exchange.fetch_ticker(pair))
    dataPrice = json.loads(r1)
  except ccxt.ExchangeError as e:
    r1 = json.dumps(exchange.fetch_ticker(pair))
    dataPrice = json.loads(r1)
  except Exception as e:
    r1 = json.dumps(exchange.fetch_ticker(pair))
    dataPrice = json.loads(r1)
  
  return (dataPrice['last'])

#Read Order To file

def readOrder():
  with open("list.txt", "r") as f:
    for line in f:
      buyRecord.append(float(line.strip()))
  print(buyRecord)

#Write Order To file

def writeOrder():
  with open("list.txt", "w") as f:       
    for ord in buyRecord:
      f.write(str(ord) +"\n")

#Get Pending Order

def showPending():
    print("Your Pending Order")
    pendingOrder = pd.DataFrame(exchange.fetch_open_orders(pair),
                   columns=['id','datetime','status','symbol','type','side','price','amount','filled','average','remaining'])
    pendingOrder.head(5)
    return pendingOrder

#Show Matched Order

def showMatched():
    print("Your Matched Order")
    matchedOrder = pd.DataFrame(exchange.fetchMyTrades(pair),
                       columns=['id','datetime', 'symbol','type','side','price','amount','cost'])
    print(matchedOrder.head(5))
    return matchedOrder

#Get Buy Matched Order

def getbuyMatched():
    matchedOrder = pd.DataFrame(exchange.fetchMyTrades(pair),
                       columns=['id','datetime', 'symbol','type','side','price','amount','cost'])
    matchedOrder = matchedOrder.loc[matchedOrder['side'] == 'buy']
    return matchedOrder

#Get Free Collatoral

def getfreeCol():
    freeCol = exchange.private_get_wallet_balances()['result'][0]['free']
    return freeCol

#Send Buy Order 
  
def sendBuy(a):
  types = 'limit'                         # ประเภทของคำสั่ง
  side = 'buy'                            # กำหนดฝั่ง BUY/SELL
  usd = a                                 # กรณี Rebalance และต้องกรอกเป็น USD
  price = getPrice() + 30                  # ระดับราคาที่ต้องการ
  size_order = round(usd/getPrice(),4)      # ใส่ขนาดเป็น BTC, ถ้า Rebalance ให้ใส่เป็น usd/price # แล้วไปกรอกในตัวแปร usd แทน
  reduceOnly = False                      # ปิดโพซิชั่นเท่าจำนวนที่มีเท่านั้น (CREDIT : TY)
  postOnly =  False                       # วางโพซิชั่นเป็น MAKER เท่านั้น
  ioc = True                            # immidate or cancel เช่น ส่งคำสั่งไป Long 1000 market 
                                          # ถ้าไม่ได้ 1000 ก็ไม่เอา เช่นอาจจะเป็น 500 สองตัวก็ไม่เอา
  ## Send Order ##
  exchange.create_order(pair, types , side, size_order, price)

  ## Show Order Status##
  print("     ")
  showPending()
  print("     ")
  showMatched()

#Send Sell Order 

def sendSell(a):
  types = 'limit'                         # ประเภทของคำสั่ง
  side = 'sell'                           # กำหนดฝั่ง BUY/SELL
  usd = a                                 # กรณี Rebalance และต้องกรอกเป็น USD
  price = getPrice() - 30                 # ระดับราคาที่ต้องการ
  size_order = round(usd/getPrice(),4)    # ใส่ขนาดเป็น BTC, ถ้า Rebalance ให้ใส่เป็น usd/price # แล้วไปกรอกในตัวแปร usd แทน
  reduceOnly = True                       # ปิดโพซิชั่นเท่าจำนวนที่มีเท่านั้น (CREDIT : TY)
  postOnly =  False                       # วางโพซิชั่นเป็น MAKER เท่านั้น
  ioc = True                              # immidate or cancel เช่น ส่งคำสั่งไป Long 1000 market 

  ## Send Order ##
  exchange.create_order(pair, types , side, size_order, price)

  ## Show Order Status##
  print("     ")
  showPending()
  print("     ")
  showMatched()

# LOGIC & Calculation SESSION

# Get Portfolio Value

def getValue():
  try:
    portValue = exchange.privateGetPositions()['result'][0]['cost']
  except:
    portValue = 0
  return portValue

# Get Zscore

def getZscore():
  zScore = round(((getPrice() - priceData.iloc[-25:-1,4].mean()) / priceData.iloc[-25:-1,4].std()),4)
  return zScore

# Get Mean Reversion

def meanRevert():

  if portValue + zScore > 1 and portValue > 0 and getPrice() > minOrder + 15 :
    sendSell()
    time.sleep(3)
    buyRecord.remove(minOrder)
    writeOrder()
    print('sell 1 USD')
    print('----------------------')
  elif portValue + zScore < -1 and freeCol > 1:
    sendBuy()
    time.sleep(3)
    matchedOrder = getbuyMatched()
    buyRecord.append(float(matchedOrder.iloc[-1: ,5]))
    writeOrder()
    print('buy 1 USD')
    print('----------------------')
  else:
    print('Current Calculation ' + str(portValue + zScore ))
    print('Waiting for Mean Reversion or minOrder at ' + str(minOrder))
    print('----------------------')
# Execute Session

while True:
  print(time.ctime())
  print('Account = ' + subaccount)
  priceData = priceHistdata()
  getPrice()
  buyRecord = []
  readOrder()
  print('Current Price = ' + str(getPrice()))
  portValue = getValue()
  print('Current Port Value = ' + str(portValue))
  x = getPrice()
  
  ### ปรับ Function บังคับตรงนี้ ###
  y = (-x + 20000) / 2000  
  
  ### Diff ระหว่าง Function บังคับ กับ Port Value
  a = y - portValue
  print('Current diff = ' + str(a))
  
  ### Function ที่ต้องการศึกษา
  actionSignal = round((priceData.iloc[-21:-1,4].mean() - 2*(priceData.iloc[-21:-1,4].std()))*2) / 2 ### Logic Function
  
  if x == actionSignal :
    if len(buyRecord) > 0 and (a > 1 or a < -1):
      if portValue > y: 
        sendSell(-a)
        time.sleep(3)
        buyRecord.remove(minOrder)
        writeOrder()
        
      elif portValue < y:
        sendBuy(a)
        matchedOrder = getbuyMatched()
        buyRecord.append(float(matchedOrder.iloc[-1: ,5]))
        writeOrder()
        
      else : 
        print('Waiting for Function Condition at ' +str(actionSignal))
    else: 
      sendBuy(a)
      matchedOrder = getbuyMatched()
      buyRecord.append(float(matchedOrder.iloc[-1: ,5]))
      writeOrder()
  else:
    print('Waiting for Function Condition at ' + str(actionSignal))
  
  time.sleep(3)
