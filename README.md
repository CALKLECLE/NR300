I build a remote control in TouchDesigner to trigger presets, update them and take photos of them.

Here you found the files:


Theres a base include parameter. By using a parameter execute:

def onValueChange(par, prev):

    if par.name == "Preset":

        pre = parent().par.Preset.eval()

        time = int(parent().par.Time.eval() * 1000)

        path = parent().par.Folderpath.eval()



        op("canon_control").module.goto_preset(pre, time)

        op("text1").par.text = "Preset " + str(pre)

        op("moviefileout1").par.file = str(path) + "/" + str(pre) + ".png"

    return



def onPulse(par):

    if par.name == "Update":

        pre = parent().par.Preset.eval()

        op("canon_control").module.update_preset(pre)



    elif par.name == "Photo":

        pre = parent().par.Preset.eval()

        path = parent().par.Folderpath.eval()

        op("moviefileout1").par.addframe.pulse()

        print(f"PTZ Preset: {pre} PNG saved under {path}")

    return

op("canon_control") is a TextDAT:

import requests

from requests.auth import HTTPBasicAuth 



ip = parent().par.Ip.eval()

user = parent().par.User.eval()

password = parent().par.Password.eval()

port = parent().par.Port.eval()



print(ip, ":", port, "/ User:", user, "/ Password:", password,)





def goto_preset(preset_num=1, time_sec=3):

    url = f"http://{ip}:{port}/-wvhttp-01-/control.cgi?p={preset_num}&p.ptztime={time_sec}"



    print("Send Request:", url)

    try:

        r = requests.get(url, auth=HTTPBasicAuth(user, password), timeout=5)

        print("State:", r.status_code)

        print("Response:", r.text)

        if r.status_code == 200:

            print(f"Preset {preset_num} → {time_sec} sec Transition OK")

        else:

            print(f"Error: HTTP {r.status_code}")

    except Exception as e:

        print("Request Error:", e)





def update_preset(preset_num=1):

    url = f"http://{ip}:{port}/-wvhttp-01-/preset/set?s1&p={preset_num}&name={preset_num}.&all=enabled"



    print("Send Request:", url)

    try:

        r = requests.get(url, auth=HTTPBasicAuth(user, password), timeout=5)

        print("State:", r.status_code)

        print("Response:", r.text)

        if r.status_code == 200:

            print(f"Preset {preset_num} save OK")

        else:

            print(f"Error: HTTP {r.status_code}")

    except Exception as e:

        print("Request Error:", e)
