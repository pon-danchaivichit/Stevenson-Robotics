    from inputs import get_gamepad
    import serial
    import threading
    import time

    class XboxController:
        MAX_TRIG_VAL = 255  # 2^8
        MAX_JOY_VAL = 32768  # 2^15

        def __init__(self):
            # Initialize controller state variables
            self.LeftJoystickY = 0
            self.LeftJoystickX = 0
            self.RightJoystickX = 0
            self.RightJoystickY = 0
            self.LeftTrigger = 0
            self.RightTrigger = 0
            self.running = True  # Flag to control the thread

            # Start the monitoring thread
            self.thread = threading.Thread(target=self._monitor_controller)
            self.thread.start()

        def read(self):
            return [self.LeftJoystickY, self.LeftJoystickX, self.RightJoystickY, self.RightJoystickX, self.LeftTrigger]

        def _monitor_controller(self):
            try:
                while self.running:
                    events = get_gamepad()
                    for event in events:
                        if event.code == 'ABS_Y':
                            self.LeftJoystickY = (event.state >>8) +128
                        elif event.code == 'ABS_X':
                            self.LeftJoystickX = (event.state >>8) +128
                        elif event.code == 'ABS_RY':
                            self.RightJoystickY = (event.state >>8) + 128
                        elif event.code == 'ABS_RX':
                            self.RightJoystickX = (event.state >>8) + 128
                        elif event.code == 'ABS_Z':
                            self.LeftTrigger = event.state
            except KeyboardInterrupt:
                # Handle any cleanup if needed
                print("Stopping controller monitoring...")

        def stop(self):
            self.running = False
            self.thread.join()

    if __name__ == '__main__':
        controller = XboxController()
        ser = serial.Serial('COM11',9600)
        time.sleep(2)
        try:
            while True:
                ser.write(bytes(controller.read()))

                print("LJS: ", str(controller.read()[1]).zfill(6), ",", str(controller.read()[0]).zfill(6), "RJS", str(controller.read()[3]).zfill(6), ",", str(controller.read()[2]).zfill(6), "LT", str(controller.read()[4]))
                time.sleep(0.1)  # Adding a small delay to avoid spamming the output
        except KeyboardInterrupt:
            controller.stop()
            print("Stopped monitoring controller.")

        
