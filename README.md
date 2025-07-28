#Name: CodeSubmissions.py
#Authors: Samantha Pintor & Selena Ouyang
#Purpose: Navigate through a maze and upon completion, spin and buzz 


from motor import Ordinary_Car
from infrared import Infrared
from adc import ADC
import buzzer
import time

#definitions
drive = Ordinary_Car()
infrared = Infrared()
buzzer = buzzer.Buzzer()
adc = ADC()
left_idr = int(adc.read_adc(0)) #left light sensor
right_idr = int(adc.read_adc(1)) #right light sensor 


LKI = 0  # Last Known Direction: 1 = Left, 2 = Right


try:

    while True:
        value = infrared.read_all_infrared()   # checks all sensors at once (0–7)

        # === LINE FOLLOWING ===
        if value == 4:  # CENTER and RIGHT sensors on line (left off)
            drive.set_motor_model(-900, -900, 600, 600)  # turn right
            LKI = 2
        elif value == 1:  # CENTER and LEFT sees line
            drive.set_motor_model(600, 600, -900, -900)  # turn left
            LKI = 1
        elif value == 6:  # only RIGHT sensor sees the line
            drive.set_motor_model(-900, -900, 600, 600)  # turn right
            LKI = 1
        elif value == 3:  # only LEFT sensor sees the line
            drive.set_motor_model(600, 600, -900, -900)  # turn left
            LKI = 2
        elif value == 5:  # only CENTER sensor sees the line
            drive.set_motor_model(-600, -600, -600, -600)  # go forward
        elif value == 7:  # All sensors off line
            print("Line lost! Using LKI =", LKI)
            if LKI == 1:
                drive.set_motor_model(-700, -700, 550, 550)  # turn right to find line
            elif LKI == 2:
                drive.set_motor_model(550, 550, -700, -700)  # turn left to find line

        print("Infrared ALL:", value, "ONE:", "LKI:", LKI)


        #LIGHT SENSOR (END DETECTION)
        
        print(f"Left Light: {left_idr}, Right Light: {right_idr}")

        if left_idr > 1.0 and right_idr > 1.0: #will only run following code when L and R reads greater than 1.0
            print("Maze end detected!")
            drive.set_motor_model(-1000, -1000, 1000, 1000)  # spin
            time.sleep(1.75) #time for one complete spin
            drive.set_motor_model(0, 0, 0, 0)  # stop
            buzzer.set_state(True) #buzzer on
            time.sleep(.1) 
            buzzer.set_state(False) #buzzer off
            break
    else:
            buzzer.set_state(False) 

except KeyboardInterrupt:
    adc.close_i2c()
    drive.set_motor_model(0, 0, 0, 0)







