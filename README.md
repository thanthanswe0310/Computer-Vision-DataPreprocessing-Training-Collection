# DWM1001C and nRF Connection Points

## Hardware Architecture
- **DWM1001C Module**: Decawave Ultra-Wideband (UWB) positioning module
- **nRF52**: Nordic nRF52 microcontroller on the DWM1001 module
- **Communication**: Primarily through nRF52 UART and BLE interfaces

## Connection Methods

### 1. UART Connection (Host API)
- **File**: [dwm1001/DWM1001_host_api/dwm1001_host_api/platform/rpi/hal/hal_uart.h](dwm1001/DWM1001_host_api/dwm1001_host_api/platform/rpi/hal/hal_uart.h)
- **Functions**: HAL_UART_Init(), HAL_UART_DeInit()
- **Purpose**: Serial communication between host (RPi/Linux) and DWM1001C module
- **Protocol**: TLV (Type-Length-Value) based

### 2. Internal nRF UART Interface
- **File**: [dwm1001/DWM1001_on_board_package/dwm/examples/dwm-uart/dwm-uart.c](dwm1001/DWM1001_on_board_package/dwm/examples/dwm-uart/dwm-uart.c)
- **Driver**: nrf_drv_uart (Nordic UART driver)
- **Instance**: NRF_DRV_UART_INSTANCE(0)
- **Event Handler**: nrf_uart_event_handler()
- **Used for**: On-board communication between nRF52 and DWM1001 UWB chip

### 3. Bluetooth Low Energy (BLE)
- **File**: [src/trilateration/protocols/ble_read.py](src/trilateration/protocols/ble_read.py)
- **Library**: Bleak (Python BLE library)
- **UUID**: location_data_uuid = "003bbdf2-c634-4b3d-ab56-7ec889b89a37"
- **Callback**: read_distance() - receives range data from DWM1001 tags
- **Purpose**: Wireless data transmission of positioning/ranging information

## Software Connection Points

### Main Thread Initialization
- **File**: [src/trilateration/tasks.py](src/trilateration/tasks.py)
- **Configuration**: [src/trilateration/commons/definitions.py](src/trilateration/commons/definitions.py)
- **Threads**:
  - BLE Discovery Thread: ble.wrap_async_ble_device_discover()
  - Serial Read Thread: uart.serial_data_read() (when enabled)
  - Data Processing: trilateration and analytics

### Data Flow
1. DWM1001C UWB chip measures distances to anchors
2. nRF52 processes ranging data via internal UART
3. Data transmitted via:
   - **BLE**: To mobile/remote clients
   - **Serial UART**: To host computer (RPi/Linux)
4. Python application parses and trilaterates position

## Current Configuration (definitions.py)
- DWM1001 Mode: Enabled (DWM1001 = 1)
- BLE_DATA: True (primary data source)
- SERIAL_DATA: False (disabled)

## Key API Functions
- `dwm_init()`: Initialize DWM1001 module
- `dwm_deinit()`: De-initialize DWM1001 module
- Low-level Module Handshake (LMH) devices coordination
