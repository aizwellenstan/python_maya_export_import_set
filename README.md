```python
# export import aiSet
import re, os, json
import maya.cmds as cmds
import pymel.core as pm
import maya.mel as mel

def export_json(filePath, data):
    with open(filePath, 'w') as f:
        f.write(json.dumps(data, sort_keys=True, indent=4 ,separators=(',', ':')))

def read_json(filePath):
    with open(filePath) as f:
        data = json.load(filePath)
        return data
    
def get_setMember(nameRegex):
    setInfo={}
    prog = re.compile(nameRegex)
    for s in pm.ls(et=pm.nt.ObjectSet):
        if prog.match(str(s)):
            attrs = [{
                'ln':attr.longName(), 
                'sn':attr.shortName(), 
                'at':attr.get(typ=True), 
                'uac':attr.isUsedAsColor(),
                'nn':pm.attributeName(attr, n=True),
                'value':attr.get()} for attr in s.listAttr(ud=True)]
            setInfo[str(s)] = {'attribute':attrs}
            setInfo[str(s)].update( {'member': [str(s) for s in s.members()]} )
    return setInfo

def rebuild_setMember(set_file):
    set_data = read_json(set_file)
    for k in set_data.keys():
        if pm.objExists(k):
            pm.delete(k)
        s = pm.sets(pm.ls(set_data[k]['member']), n=k)
        for attr in set_data[k]['attribute']:
            value = attr.pop('value')
            pm.addAttr(s, **attr)
            s.attr(attr['ln']).set(value)

set_data = get_setMember('set_*')


filePath = r'D:\set_member.json'

#export set
export_json(filePath, set_data)

#import set
#rebuild_setMember(filePath)
```
