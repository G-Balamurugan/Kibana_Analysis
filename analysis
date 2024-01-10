import json
from prettytable import PrettyTable
import matplotlib.pyplot as plt

# Invertal to Filter 
inTime = "2024-01-03T16:51:30.481245" 
outTime = "2024-01-10T20:40:30.481245"

logData = []
logCount , finalLogData , visualizeData , resData = {} , {} , {} , {"TotalCount": 0}
logInputFile = "/Users/balamurugan/Desktop/KibanaAnalysis/LogFile/sample.json"
logReportFile = "/Users/balamurugan/Desktop/KibanaAnalysis/LogFile/kibana_report_result.json"
logVisualizeFile = "/Users/balamurugan/Desktop/KibanaAnalysis/LogFile/visualize.json"

# def extractTXID(transId): return transId.index("tx.id=")
# tx.id is always at the index -> '41'
def extractEventName(trans):
    ind , eventName , eventNameTime , updateType = 47 , "" , "" , ""
    while(trans[ind] != "-"):
       eventNameTime += trans[ind]
       ind += 1

    eventName , eventNameTime = eventNameTime , eventNameTime + trans[ind]
    ind += 1
    
    while(trans[ind] != " "):
        eventNameTime += trans[ind]
        ind += 1
    
    ind = trans.find("updateThrottler")
    if ind != -1:
       indUpdate = ind + trans[ind:].find("updateType:")
       indUpdate += 12
       while(trans[indUpdate] != ","):
          updateType += trans[indUpdate]
          indUpdate += 1

    return [eventName.strip(), eventNameTime.strip(), updateType.strip()]

# Input Log File Read
with open(logInputFile) as userFile:
  fileContents = userFile.read()
fileContents = json.loads(fileContents)

data = fileContents["hits"]["hits"]
for element in data:
  logData.append(element["_source"]["log"])

for log in logData:
    eventName, transId, eventUpdate = extractEventName(log)

    if eventName not in finalLogData:
        finalLogData[eventName] = {}
    if transId not in finalLogData[eventName]:
        finalLogData[eventName][transId] = {"updateType": "", "normalLog": [], "updateLog": {"retailCollection" : [] , "recommendation" : []}}

    if eventUpdate != "":
        finalLogData[eventName][transId]["updateType"] = eventUpdate
        if log.find("retailCollection") != -1: finalLogData[eventName][transId]["updateLog"]["retailCollection"].append(log)
        else: finalLogData[eventName][transId]["updateLog"]["recommendation"].append(log)
    else:
        finalLogData[eventName][transId]["normalLog"].append(log)

for event in finalLogData:
    finalLogData[event] = dict(sorted(finalLogData[event].items()))

    eachTrans = {}
    for trans in finalLogData[event]:
        retailCount = len(finalLogData[event][trans]["updateLog"]["retailCollection"])
        l5Count = len(finalLogData[event][trans]["updateLog"]["recommendation"])
        eachTrans[trans] = {
            "timeStamp" : trans[-26:],
            "normalLogCount" : len(finalLogData[event][trans]["normalLog"]),
            "updateType" : finalLogData[event][trans]["updateType"],
            "updateLogCount" : retailCount + l5Count,
            "retailCollectionCount" : retailCount,
            "recommendationCount" : l5Count   
        }
    visualizeData[event] = {
        "logCount" : len(finalLogData[event]),
        "trans" : eachTrans
    }
   
# Report File Entry 
finalLogData = json.dumps(finalLogData, indent=4)
with open(logReportFile, "w") as outfile:
    outfile.write(finalLogData)

for event in visualizeData:
    event_count = 0
    for trans in visualizeData[event]["trans"]:
        if trans[-26:] >= inTime and trans[-26:] <= outTime:
            event_count += visualizeData[event]["trans"][trans]["normalLogCount"]
            event_count += visualizeData[event]["trans"][trans]["updateLogCount"]
    resData[event] = event_count
    resData["TotalCount"] += event_count

print("Between the Interval ... " , inTime , " -> " , outTime , "\n")
visualizeDataStr = json.dumps(dict(sorted(resData.items() , key=lambda item: item[1] ,reverse=True)) , indent=4)
print(visualizeDataStr)

# VisalizeData File Entry
visualizeData = dict(sorted(visualizeData.items(), key=lambda x: x[1]["logCount"], reverse=True))
visualizeData = json.dumps(visualizeData , indent=4)
with open(logVisualizeFile, "w") as outfile:
    outfile.write(visualizeData)

# Render Bar Graph Event Occurrence
eventsList = list(resData.keys())
countsList = list(resData.values())

plt.figure(figsize=(10, 10))
bars = plt.bar(eventsList, countsList, color='skyblue')
plt.xlabel('Events')
plt.ylabel('Count')
plt.title('Event Counts')
plt.xticks(rotation=45, ha='right')  # Rotate x-axis labels for better readability
plt.tight_layout()
plt.title("Between the Interval ... " + inTime + " => " + outTime)  # Title for the entire bar graph
for bar, count in zip(bars, countsList):    # Adding actual count on top of each bar
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.1, f"{count}", ha='center', va='bottom')

plt.show()  # Show or save the plot
