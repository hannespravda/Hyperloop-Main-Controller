**Instructions for MainController**

General Information: https://wiki.next-prototypes.de/display/next/BJA+Pod+IPC+Software

DataHandling:
	all incoming and outgoing variables are stored in STRUCTS in the GVL OperationControlData or XMCData
	variables derive from the interface tables at https://wiki.next-prototypes.de/display/next/Interfaces
	all data is written to internal variables inside methode doDatahandling (PRG_MainController)
	Sub Level Systems require variables avvording to their INPUTS/OUTPUTS
	High Level Systems forward data from or to Sub Level Systems via Properties
	Top Level doesnt write data directly to Sub Level systems
	Adding new variables:
		add variable to STRUCT (Type System --> DataTypes --> search STRUCT e.g. FromMainControlToOperationControl --> Edit)
		Add property to High Level System (Input = Set, Output = Get) --> compare to other properties
		Add variable to Sub Level system Inputs/Outputs
		Write variable from GVL to High Level Property in doDataHandling
		
ErrorHandling:
	all Sub Level Systems contain their individual error as ErrorBist locally defines
	High Level systems collect nErrorID of Sub Level
	Top Level ErrorHandling collects nErrorIDs from High Level
	Add new Error Bits:
		Add Error BOOl in Sub Level system
		write Error Bit to Sub Level nErrorID
		update High Level nErrorID
		update doSetWarningBits or doSetErrorBits in ErrorHandling
		for EventLogging add Error Message in FB_EventLogging
		update number of Warnings/ Errors/ Infos in PRG_ErrorHandling
		
DebugMode:
	If nDebugMode is set, startup check is skipped and errors do not cause a transition to ERROR
	errors and warnings still occur in the event logging
	apart from STARTUP check, all function blocks are fully functional
	Use only in testing, if it is assured that errors do not cause harm to hardware
	
Dummy Methods:
	TrajectoryGenerator and GuidanceController are implemented as dummies
	They do not contain functionality yet!!!
	they are provided with the inputs and outputs according to the declaration
	nOperationMode is forwarded to the XMCBoards as well stored in the High Level system to set as an internal input
	the function blocks contain methods which describe the behaviour, but they are empty
	replace the methods OR put the logic of the control structures inside the methods (you can adapt the respective inputs)
	If you need different/ more/ less inputs update the DataHandling
	Dummy Models can work in different operation Modes according to nOperationMode (if not needed, only forward to XMCBoards)
	Total levitation Height given back to OperationControl and XMCBoards is ActualLevDistance + EqualizingControl OutputVector
	TrajectoryGenerator should contain predefined values for a trajectory back to ground level which can be called in SHUTDOWN
		--> SHUTDOWN in case of error can be performed automatically
	
Hardware Setup:
	EL6001:
			Baudrate 115200
			8N1 Data Frame
			other Settings all FALSE
			SerialCom Library
			absolutetly make sure to bridge Ground from Coupler to Clamp (otherwise only 255 as values)
	Master:
		x300 upper port connected to OperationController/ Laptop
		Laptop (TwinCAT running on Local)
			add manually EtherCAT Master
			Scan in Config Mode --> add CX2040 as Slave
			after adding IOs to Slave, delete CX2040 and scan again
	Slave:
		to activate configuration: connect x001 port to laptop -> important: check adapter settings
		TwinCAT running on CX2040 (target system)
			IP Adresse CX2040: 169.254.246.211, password: 1, User: Administrator 
				--> Ethernet Adapter des Laptops: 169.254.246.xxx
		Coupler connected to X000 port
		Scan in Config Mode and add EtherCAT slave (CX2040) and EtherCAT device (Coupler)
		Connect Clamp IOs (EL6001, EL6021, EL2564) to GVL structs (should recognise fitting variable automatically)
		Add manually IO STRUCTS to Slave Inputs Outputs
			add new element
			search for STRUCT (e.g. FromMainControlToOperationControl
			connect to GVL (the whole STRUCT)
			update Slave at EtherCAT Master TwinCAT (delete and scan again)