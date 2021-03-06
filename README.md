# vanet_simulation
vanet simulation using ns3 and sumo

//Real time simulation of Vehicular Ad-Hoc Networks (VANET) using NS3 and SUMO

	WAVE- Wireless Access for Vehicular Environments

//Tools required
	Ubuntu 16.04 LTS

	Ns3 version: ns-3.29

	SUMO version: Eclipse SUMO Version v1_5_0+1154-40cc386

	Open street Map (osmWebWizard.py)


//step 1:open osmWebWizard.py

	$ cd sumo/tools

	$ python osmWebWizard.py

//step 2: Process and find osm.sumocfg and generate vanettrace.xml

	$ cd sumo/tools/2022-06-07-11-58-26/

	find osm.sumocfg

	$ sumo -c osm.sumocfg  --fcd-output vanettrace.xml


//step 3: find traceExporter.py and generate vanetmobility.tcl
	$ cd sumo/tools

	//find traceExporter.py

	$ python traceExporter.py -i 2022-06-07-11-58-26/trace.xml --ns2mobility-		
	output=2022-06-07-11-58-26/vanetmobility.tcl
	//Find total nodes in vanetmobility.tcl
	
//Step 4: Add netanimation file

	#include "ns3/netanim-module.h"

	AnimationInterface anim("Vanetanim.xml"); //Add before Simulator::Run();

//Step 5: Add performance analysis code to vanet program

///////////////////////// Network Perfomance Calculation ///////////////////////
uint32_t SentPackets = 0;
uint32_t ReceivedPackets = 0;
uint32_t LostPackets = 0;
int j=0;
float AvgThroughput = 0;
Time Jitter;
Time Delay;

Ptr<Ipv4FlowClassifier> classifier = DynamicCast<Ipv4FlowClassifier> (flowmon.GetClassifier ());
  std::map<FlowId, FlowMonitor::FlowStats> stats = monitor->GetFlowStats ();

  for (std::map<FlowId, FlowMonitor::FlowStats>::const_iterator iter = stats.begin (); iter != stats.end (); ++iter)
    {
	  Ipv4FlowClassifier::FiveTuple t = classifier->FindFlow (iter->first);

NS_LOG_UNCOND("----Flow ID:" <<iter->first);
NS_LOG_UNCOND("Src Addr" <<t.sourceAddress << "Dst Addr "<< t.destinationAddress);
NS_LOG_UNCOND("Sent Packets=" <<iter->second.txPackets);
NS_LOG_UNCOND("Received Packets =" <<iter->second.rxPackets);
NS_LOG_UNCOND("Lost Packets =" <<iter->second.txPackets-iter->second.rxPackets);
NS_LOG_UNCOND("Packet delivery ratio =" <<iter->second.rxPackets*100/iter->second.txPackets << "%");
NS_LOG_UNCOND("Packet loss ratio =" << (iter->second.txPackets-iter->second.rxPackets)*100/iter->second.txPackets << "%");
NS_LOG_UNCOND("Delay =" <<iter->second.delaySum);
NS_LOG_UNCOND("Jitter =" <<iter->second.jitterSum);
NS_LOG_UNCOND("Throughput =" <<iter->second.rxBytes * 8.0/(iter->second.timeLastRxPacket.GetSeconds()-iter->second.timeFirstTxPacket.GetSeconds())/1024<<"Kbps");

SentPackets = SentPackets +(iter->second.txPackets);
ReceivedPackets = ReceivedPackets + (iter->second.rxPackets);
LostPackets = LostPackets + (iter->second.txPackets-iter->second.rxPackets);
AvgThroughput = AvgThroughput + (iter->second.rxBytes * 8.0/(iter->second.timeLastRxPacket.GetSeconds()-iter->second.timeFirstTxPacket.GetSeconds())/1024);
Delay = Delay + (iter->second.delaySum);
Jitter = Jitter + (iter->second.jitterSum);

j = j + 1;

}

AvgThroughput = AvgThroughput/j;
NS_LOG_UNCOND("--------Total Results of the simulation----------"<<std::endl);
NS_LOG_UNCOND("Total sent packets  =" << SentPackets);
NS_LOG_UNCOND("Total Received Packets =" << ReceivedPackets);
NS_LOG_UNCOND("Total Lost Packets =" << LostPackets);
NS_LOG_UNCOND("Packet Loss ratio =" << ((LostPackets*100)/SentPackets)<< "%");
NS_LOG_UNCOND("Packet delivery ratio =" << ((ReceivedPackets*100)/SentPackets)<< "%");
NS_LOG_UNCOND("Average Throughput =" << AvgThroughput<< "Kbps");
NS_LOG_UNCOND("End to End Delay =" << Delay);
NS_LOG_UNCOND("End to End Jitter delay =" << Jitter);
NS_LOG_UNCOND("Total Flod id " << j);
monitor->SerializeToXmlFile("manet-routing.flowmon", true, true);

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

 
//step 6:configure scenario 2

else if (m_scenario == 2)
 {
      // Realistic vehicular trace in KCT
      // "low density, 21 total vehicles"
      m_traceFile = "/home/Sharat /Desktop/adhoc/sumo/tools/2022-06-07-11-58-26/vanetmobility.tcl";
      m_logFile = "vanet.log";
      m_mobility = 1;
      m_nNodes = 21;
      m_TotalSimTime = 150.01;
      m_nodeSpeed = 0;
      m_nodePause = 0;
      m_CSVfileName = "vanet.csv";
      m_CSVfileName = "vanet2.csv";
    }

//Step 7:Run the program

	$ ./waf --run "scratch/vanet-routing-compare --protocol=1 --scenario=2"



//Step 8: Open Netanim
 	$ cd ns-allinone-3.29/netanim
 	$ ./NetAnim
 
	Load Vanetanim.xml file and see the network animation
