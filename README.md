# BI
import os
import re
import sys
import numpy
import shutil
import logging
import sqlalchemy
import numpy as np
import pandas as pd
import urllib

from datetime import datetime
from sqlalchemy import create_engine, MetaData, Table, select
def Routine():
   
    MT202_traité = 'E:/BISWIFT/Traité/MT200+MT202/Success'
    MT202_non_traité = 'E:/BISWIFT/Traité/MT200+MT202/Failure'

    path = 'E:/BISWIFT'
    
    global fichier
    
    # Create and configure logger
    logging.basicConfig(filename="E:/BISWIFT/LogFile_MT202.log",format='%(asctime)s %(message)s',filemode='w')
    # Creating an object
    logger = logging.getLogger()
 
    # Setting the threshold of logger to DEBUG
    logger.setLevel(logging.DEBUG)
    
    # Création de connection MSSQL:

    params = urllib.parse.quote_plus("DRIVER={SQL Server};SERVER=DESKTOP-SU89BDQ;DATABASE=Swift")
    engine = sqlalchemy.create_engine("mssql+pyodbc:///?odbc_connect=%s" % params, connect_args={'timeout': 30})


    engine.connect() 
    logger.info(" \n ")
    logger.info(140*"_")
    logger.info(" Connexion avec la BDD ") 
    
    # Test messages
    logger.info(" Début de traitement !")
    logger.info(" \n ")
    
    
    

    for root, directories, files in os.walk(path):  
        for file in files:
            fichier = files

        #Date systeme:
        def date():
            d = datetime.now()
            date = d.strftime("%d-%m-%Y %H:%M:%S")

            return(date)

        date = date()
        
        i=0
        for i in range(len(fichier)):
            a = path + '/' + fichier[i]

            basename = os.path.basename(a)
            file_name = os.path.splitext(basename)[0]

            y = str( file_name )

            #Lecture des données:
            def data():

                fichier = open(a, 'r')
                data = fichier.read().replace('\n', ' ').replace(',', '.').replace('\x01', '').replace('$','\x03')
                fichier.close()

                return(data)

            data = data()

            ec = data.count(':13C:')
            eh = data.count('{1:F01')
            
            if '{2:O202' in data :
                

                if ec > 1 :
                    shutil.move(os.path.join(path,a),MT202_non_traité)

                else : 
                    
                    if eh == 1 :
                        try : 
                        


                            Decompose = data.replace("{1:F21", "F21").replace("}{1:F01", ",F01").replace("{1:F01", "F01").replace("}{2:O",",O").replace("{3:{108:",",{").replace("{3:{111:",",{").replace("{3:{121:",",{").replace("{3:{121:",",{").replace("{121:",",{").replace("}}{4:", "}}").replace("}{4:{177:",",{177:").replace(':20:',',').replace(":21:", ",").replace(":13C:", ",").replace(":32A:",",").replace(":52A:",",").replace(":52D:",",").replace(":53A:",",").replace(":53B:",",").replace(":53C:",",").replace(":54A:",",").replace(":54B:",",").replace(":54D:",",").replace(":56A:",",").replace(":56D:",",").replace(":57A:",",").replace(":57B:",",").replace(":57D:",",").replace(":58A:",",").replace(":58D:",",").replace(":72:",",").replace('{5:',',{').replace('{S:',',{').replace("-}","").split(',')

                            logger.info("\n" + 40*"-" + fichier[i]+ 40*"-" )
                            logger.info(" Décomposition du message ")

                            def DataFrame():
                                #Définition du DataFrame:
                                b = pd.DataFrame.from_dict(Decompose)
                                
                                c = b.T
                                
                                return(c)
                            
                            def preparation():
                                #Détermination des colonnes:
                                column_names = []
                                column_names.clear()

                                L = str(Decompose)
                                

                                if '{1:F21' in data :
                                   
                                    column_names.append('Block01_ACK')
                                    
                                if '{177:' in data :
                                    column_names.append('Text_bloc')
                                if '{1:F01' in data :
                                    column_names.append('Block01_FIN')
                                if '{2:I202' in data :
                                    column_names.append('Block02_Application_Header_input')
                                if '{2:O202' in data :
                                    column_names.append('Block02_Application_Header_output')
                                if '{3:{108:' in data :
                                    column_names.append('Header_Block')
                                if '{3:{111:' in data :
                                    column_names.append('Header_Block')

                                if '{121' in data :
                                        column_names.append('UETR')

                                if ':20:' in data :
                                    column_names.append('20_Transaction_ref')

                                if ':21:' in data :
                                    column_names.append('21_Operation_code')


                                if ':13C:' in data : 
                                    column_names.append('13C_Time_indication')

                                if ':32A:' in data :
                                    column_names.append('32A_Settled_amount')

                                if ':52A:' in data :
                                    column_names.append('52A_Ordering_institution')
                                if ':52D:' in data :
                                    column_names.append('52D_Ordering_institution')

                                if ':53A:' in data :
                                    column_names.append('53_Sender_correspondent')
                                if ':53B:' in data :
                                    column_names.append('53_Sender_correspondent')
                                if ':53D:' in data :
                                    column_names.append('53_Sender_correspondent')

                                if ':54A:' in data :
                                    column_names.append('54_Receiver_correspondent')
                                if ':54B:' in data :
                                    column_names.append('54_Receiver_correspondent')
                                if ':54D:' in data :
                                    column_names.append('54_Receiver_correspondent')


                                if ':56A:' in data :
                                    column_names.append('56A_Intermediary')
                                if ':56D:' in data :
                                    column_names.append('56D_Intermediary')

                                if ':57A:'  in data :
                                    column_names.append('57A_Account')
                                if ':57B:' in data :
                                    column_names.append('57B_Account')
                                if ':57D:' in data :
                                    column_names.append('57D_Account')

                                if ':58A:' in data :
                                    column_names.append('58A_Beneficiary')
                                if ':58D:' in data :
                                    column_names.append('58D_Beneficiary')

                                if ':72:' in data :
                                    column_names.append('72_Info')


                                if '{5:' in data :
                                    column_names.append('Trailer_Block')
                                if '{S:' in data :
                                    column_names.append('System_Trailer_Block')

                                    
                                return(column_names)
                                

                            c = DataFrame()
                            c.columns = preparation()

                            
                            def parse_first21_block(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r"{1:F21(\w{12})(\d{4})(\d+)}"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 3:
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                    labeled_entries.append(block_entries[2])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                return(labeled_entries)

                            def parse_first01_block(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r"{1:F01(\w{12})(\d{4})(\d+)}"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 3:
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                    labeled_entries.append(block_entries[2])
                                return(labeled_entries)

                            def parse_2nd_block_O(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r'{2:O(\d{3})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})(\w{12})(\d{4})(\d{6})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})(\w{1})'

                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))
                                if len(block_entries) >= 15:
                                    labeled_entries.append('Arrivé')
                                    labeled_entries.append( block_entries[0])
                                    labeled_entries.append( block_entries[5] + '-' + block_entries[4] + '-' + block_entries[3] + ' ' + block_entries[1] + ':' + block_entries[2] + ':00')
                                    labeled_entries.append( block_entries[6])
                                    labeled_entries.append( block_entries[7])
                                    labeled_entries.append( block_entries[8])
                                    labeled_entries.append( block_entries[11] + '-' + block_entries[10] + '-' + block_entries[9] + ' ' + block_entries[12] + ':' + block_entries[13] + ':00')
                                    if block_entries[14] == 'N' :
                                        labeled_entries.append('Normal')
                                    elif block_entries[14] =='S' :
                                        labeled_entries.append('System')
                                    elif block_entries[14] =='U' :
                                        labeled_entries.append('Urgent')
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                return(labeled_entries)

                            def parse_Set_Amount(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":32A:(\d{2})(\d{2})(\d{2})(\w{3})(\d+\.\d*)"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 5 :
                                    labeled_entries.append( block_entries[2] + '-' + block_entries[1] + '-' + block_entries[0] )
                                    labeled_entries.append(block_entries[3])
                                    labeled_entries.append(block_entries[4])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                return(labeled_entries)
                            
                            def parse_52A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":52A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)


                            def parse_52D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":52D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            
                            def parse_56A(msg):
                                block_entries = []
                                labeled_entries = []
                                
                                regex = r":56A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)

                            def parse_56D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":56D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            
                            def parse_57A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":57A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"

                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    #labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            def parse_57D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":57D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            
                            def parse_58A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":58A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >=1  :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])

                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    
                                return(labeled_entries)

                            def parse_58D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":58D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)


                                return(labeled_entries)

                            def app_21(message_text):
                                mt_message_21 = []

                                try:

                                    if message_text is not None:
                                        mt_message_21.append(parse_first21_block(message_text))
                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_21

                            def app_01(message_text):
                                mt_message_01 = []

                                try:

                                    if message_text is not None:
                                        mt_message_01.append(parse_first01_block(message_text))
                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_01

                            def app_02_O(message_text):
                                mt_message = []

                                try:

                                    if message_text is not None:
                                        mt_message.append(parse_2nd_block_O(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message

                            def app_Set_Amount(message_text):
                                mt_message_SA = []

                                try:

                                    if message_text is not None:
                                        mt_message_SA.append(parse_Set_Amount(message_text))
                                    else : 
                                        mt_message_SA.append(parse_Set_Amount(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_SA
                                
                            def app_52A(message_text):
                                mt_message_52A = []

                                try:

                                    if message_text is not None:
                                        mt_message_52A.append(parse_52A(message_text))
                                    else : 
                                        mt_message_52A.append(parse_52A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_52A

                            def app_52D(message_text):
                                mt_message_52D = []

                                try:

                                    if message_text is not None:
                                        mt_message_52D.append(parse_52D(message_text))
                                    else : 
                                        mt_message_52D.append(parse_52D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_52D
                                
                            def app_56A(message_text):
                                mt_message_56A = []

                                try:

                                    if message_text is not None:
                                        mt_message_56A.append(parse_56A(message_text))
                                    else : 
                                        mt_message_56A.append(parse_56A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_56A
                            def app_56D(message_text):
                                mt_message_56D = []

                                try:

                                    if message_text is not None:
                                        mt_message_56D.append(parse_56D(message_text))
                                    else : 
                                        mt_message_56D.append(parse_56D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_56D
                            def app_57A(message_text):
                                mt_message_57A = []

                                try:

                                    if message_text is not None:
                                        mt_message_57A.append(parse_57A(message_text))
                                    else : 
                                        mt_message_57A.append(parse_57A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_57A
                                
                            def app_57D(message_text):
                                mt_message_57D = []

                                try:

                                    if message_text is not None:
                                        mt_message_57D.append(parse_57D(message_text))
                                    else : 
                                        mt_message_57D.append(parse_57D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_57D

                            def app_58A(message_text):
                                mt_message_58A = []

                                try:

                                    if message_text is not None:
                                        mt_message_58A.append(parse_58A(message_text))
                                    else : 
                                        mt_message_58A.append(parse_58A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_58A
                            
                            def app_58D(message_text):
                                mt_message_58D = []

                                try:

                                    if message_text is not None:
                                        mt_message_58D.append(parse_58D(message_text))
                                    else : 
                                        mt_message_58D.append(parse_58D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_58D

                            w_res = app_21(data)

                            x_res = app_01(data)
                            y_res_I = app_02_O(data)

                            sa_res = app_Set_Amount(data)
                            
                            Ordering_A = app_52A(data)
                            Ordering_D = app_52D(data)
                            Intermediary_A = app_56A(data)
                            Intermediary_D = app_56D(data)
                            Account_A = app_57A(data)
                            Account_D = app_57D(data)
                            
                            Beneficiary_A = app_58A(data)
                            Beneficiary_D = app_58D(data)

                            x_21 = pd.DataFrame.from_dict(w_res)
                            
                            x_01 = pd.DataFrame.from_dict(x_res)
                            y_I = pd.DataFrame.from_dict(y_res_I)

                            sa = pd.DataFrame.from_dict(sa_res)
                            
                            O_A = pd.DataFrame.from_dict(Ordering_A)
                            O_D = pd.DataFrame.from_dict(Ordering_D)
                            I_A = pd.DataFrame.from_dict(Intermediary_A)
                            I_D = pd.DataFrame.from_dict(Intermediary_D)
                            A_A = pd.DataFrame.from_dict(Account_A)
                            A_D = pd.DataFrame.from_dict(Account_D)
                            
                            B_A = pd.DataFrame.from_dict(Beneficiary_A)
                            B_D = pd.DataFrame.from_dict(Beneficiary_D)

                            x_21.columns = ['Receiver_BIC_F21','Session_Nbr_F21','Sequence_Nbr_F21']
                            x_01.columns = ['Receiver_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01']
                            y_I.columns = ['Sens_Swift','Msg_Type','In_time_date','Sender_BIC_CODE','Session','Isn','Out_time_date','Msg_Priority']

                            sa.columns = ['ValueDate', 'Currency', 'Amount']
                            
                            O_A.columns = ['52A_Party_Identifier','52A_Identifier_Code']
                            O_D.columns = ['52D_Party_Identifier','52D_Name_Address']
                            I_A.columns = ['56A_Party_Identifier','56A_Identifier_Code']
                            I_D.columns = ['56D_Party_Identifier','56D_Name_Address']
                            A_A.columns = ['57A_Party_Identifier','57A_Identifier_Code']
                            A_D.columns = ['57D_Party_Identifier','57D_Name_Address']

                            B_A.columns = ['58A_Party_Identifier','58A_Identifier_Code']
                            B_D.columns = ['58D_Party_Identifier','58D_Name_Address']
                            
                            R = pd.DataFrame.from_dict(x_01['Receiver_BIC_F01'])
                            
                            R.columns = ['Receiver_BIC_Code']
                            
                            df = pd.read_sql_table('Banques_Correspondantes_Vostro', con=engine)
                            
                            def search():
                                s = 0
                                for name in df.BIC:
                                    start = 0
                                    stop = len(y_I.Sender_BIC_CODE[0][:8])
                                    found = y_I.Sender_BIC_CODE[0][:8].find(name[:8], start, stop)
                                    

                                    if found == 0 : 
                                        s=1
                                return s
                            search = search()
                            

                            ss = pd.append([x_21,x_01, y_I, sa,O_A,O_D,I_A,I_D,A_A,A_D, B_A, B_D, R ], axis=1, join='inner')
                            print(ss)
                            k = [date] + [file_name] + [data]
                            
                            l = pd.DataFrame.from_dict(k)
                            m = l.T
                            m.columns = ['Treatment_Date','File_Name','Swift_Message']

                            u = pd.append([c, ss, m], axis=1, join='inner')

                            att =['Receiver_BIC_F21', 'Session_Nbr_F21', 'Sequence_Nbr_F21','Receiver_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01','Sens_Swift','Msg_Type','In_time_date','Sender_BIC_CODE','Etat','Session','Isn','Out_time_date','Msg_Priority','Block01_ACK','Text_bloc','Block01_FIN','Block02_Application_Header_input','Block02_Application_Header_output','Header_Block','UETR','20_Transaction_ref','21_Operation_code','13C_Time_indication','32A_Settled_amount','ValueDate', 'Currency', 'Amount','52A_Ordering_institution','52A_Party_Identifier','52A_Identifier_Code','52D_Ordering_institution','52D_Party_Identifier','52D_Name_Address','53_Sender_correspondent','54_Receiver_correspondent','56A_Intermediary','56A_Party_Identifier','56A_Identifier_Code','56D_Intermediary','56D_Party_Identifier','56D_Name_Address','57A_Account','57A_Party_Identifier','57A_Identifier_Code','57D_Account','57D_Party_Identifier','57D_Name_Address','58A_Beneficiary','58A_Party_Identifier','58A_Identifier_Code','58D_Beneficiary','58D_Party_Identifier','58D_Name_Address','72_Info','Trailer_Block','System_Trailer_Block','Treatment_Date','File_Name','Swift_Message']
                            all = pd.DataFrame(columns = att)

                            tab = all.append(u, ignore_index = True)
                          
                            tab['Etat'] = np.where(search==1, 'A traiter','A ne pas traiter')

                            tab.to_sql(name='202',con=engine, index=False, if_exists='append')

                            shutil.move(os.path.join(path,a),MT202_traité)

                            logger.info(" Ligne ajoutée à la base de données avec succès")
                            logger.info(" Redirection du fichier " + "'" + fichier[i] + "'" + " vers " + MT202_traité)

                            logger.info(" Fin du traitement du fichier " + "'" + fichier[i] + "'")
                            
                        
                        except : 
                            #raise
                            pass
                        
#________________________________________________________Output_Rép________________________________________________


                    elif eh > 1 :
                        
                        ddd = data.split('\x03')
                        lgr = len(ddd)
                        op = 0
                            
                        for op in range(lgr):
                            
                            try : 
                                Decompose = ddd[op].replace("{1:F21", "F21").replace("}{1:F01", ",F01").replace("{1:F01", "F01").replace("}{2:O",",O").replace("{3:{108:",",{").replace("{3:{111:",",{").replace("{3:{121:",",{").replace("{121:",",{").replace("}}{4:", "}}").replace("}{4:{177:",",{177:").replace(':20:',',').replace(":21:", ",").replace(":13C:", ",").replace(":32A:",",").replace(":52A:",",").replace(":52D:",",").replace(":53A:",",").replace(":53B:",",").replace(":53C:",",").replace(":54A:",",").replace(":54B:",",").replace(":54D:",",").replace(":56A:",",").replace(":56D:",",").replace(":57A:",",").replace(":57B:",",").replace(":57D:",",").replace(":58A:",",").replace(":58D:",",").replace(":72:",",").replace('-}{5:',',{').replace('{S:',',{').replace("-}","").split(',')
                                
                                logger.info("\n" + 40*"-" + fichier[i]+ 40*"-" )
                                logger.info(" Décomposition du message N°" + str(op))
                                #if '{2:I103' or '{2:O103' in ddd[op]:
                                if not '{2:O202' in ddd[op]:
                                    logger.info(" Redirection du message vers " + path + " sous le nom de " + file_name + '_' + str(op) + '.txt')
                                    with open(path + '/' + file_name + '_' + str(op) + '.txt', 'x') as f:
                                        f.write(ddd[op])
                                    #shutil.copy(os.path.join(path,a),MT202_non_traité)

                                def DataFrame():
                                    #Définition du DataFrame:
                                    b = pd.DataFrame.from_dict(Decompose)
                                    c = b.T

                                    return(c)

                                def preparation():
                                    #Détermination des colonnes:
                                    L = str(Decompose)

                                    column_names = []
                                    column_names.clear()

                                    if '{1:F21' in ddd[op] :
                                        print('a')
                                        column_names.append('Block01_ACK')
                                    if '{177:' in ddd[op] :
                                        column_names.append('Text_bloc')
                                    if '{1:F01' in ddd[op] :
                                        column_names.append('Block01_FIN')
                                    if '{2:I202' in ddd[op] :
                                        column_names.append('Block02_Application_Header_input')
                                    if '{2:O202' in ddd[op] :
                                        column_names.append('Block02_Application_Header_output')
                                    if '{3:{108:' in ddd[op] :
                                        column_names.append('Header_Block')
                                    if '{3:{111:' in ddd[op] :
                                        column_names.append('Header_Block')
                                    if '{121' in ddd[op] :
                                        column_names.append('UETR')

                                    if ':20:' in ddd[op] :
                                        column_names.append('20_Transaction_ref')

                                    if ':21:' in ddd[op] :
                                        column_names.append('21_Operation_code')

                                    if ':13C:' in ddd[op] : 
                                        column_names.append('13C_Time_indication')

                                    if ':32A:' in ddd[op] :
                                        column_names.append('32A_Settled_amount')

                                    if ':52A:' in ddd[op] :
                                        column_names.append('52A_Ordering_institution')
                                    if ':52D:' in ddd[op] :
                                        column_names.append('52D_Ordering_institution')

                                    if ':53A:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')
                                    if ':53B:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')
                                    if ':53D:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')

                                    if ':54A:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')
                                    if ':54B:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')
                                    if ':54D:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')

                                    if ':56A:' in ddd[op] :
                                        column_names.append('56A_Intermediary')
                                    if ':56D:' in ddd[op] :
                                        column_names.append('56D_Intermediary')

                                    if ':57A:'  in ddd[op] :
                                        column_names.append('57A_Account')
                                    if ':57B:' in ddd[op] :
                                        column_names.append('57B_Account')
                                    if ':57D:' in ddd[op] :
                                        column_names.append('57D_Account')


                                    if ':58A:' in ddd[op] :
                                        column_names.append('58A_Beneficiary')
                                    if ':58D:' in ddd[op] :
                                        column_names.append('58D_Beneficiary')

                                    if ':72:' in ddd[op] :
                                        column_names.append('72_Info')

                                    if '{5:' in ddd[op] :
                                        column_names.append('Trailer_Block')
                                    if '{S:' in ddd[op] :
                                        column_names.append('System_Trailer_Block')

                                    return (column_names)

                                c = DataFrame()
                                c.columns = preparation()

                                def parse_first21_block(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r"{1:F21(\w{12})(\d{4})(\d+)}"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 3:
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                        labeled_entries.append(block_entries[2])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                    return(labeled_entries)

                                def parse_first01_block(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r"{1:F01(\w{12})(\d{4})(\d+)}"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 3:
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                        labeled_entries.append(block_entries[2])
                                    return(labeled_entries)

                                def parse_2nd_block_O(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r'{2:O(\d{3})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})(\w{12})(\d{4})(\d{6})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})(\w{1})'

                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))
                                    if len(block_entries) >= 15:
                                        labeled_entries.append('Arrivé')
                                        labeled_entries.append( block_entries[0])
                                        labeled_entries.append( block_entries[5] + '-' + block_entries[4] + '-' + block_entries[3] + ' ' + block_entries[1] + ':' + block_entries[2] + ':00')
                                        labeled_entries.append( block_entries[6])
                                        labeled_entries.append( block_entries[7])
                                        labeled_entries.append( block_entries[8])
                                        labeled_entries.append( block_entries[11] + '-' + block_entries[10] + '-' + block_entries[9] + ' ' + block_entries[12] + ':' + block_entries[13] + ':00')
                                        if block_entries[14] == 'N' :
                                            labeled_entries.append('Normal')
                                        elif block_entries[14] =='S' :
                                            labeled_entries.append('System')
                                        elif block_entries[14] =='U' :
                                            labeled_entries.append('Urgent')
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                    return(labeled_entries)

                                def parse_Set_Amount(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r':32A:(\d{2})(\d{2})(\d{2})(\w{3})(\d+\.\d*)'
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 5 :
                                        labeled_entries.append( block_entries[2] + '-' + block_entries[1] + '-' + block_entries[0] )
                                        labeled_entries.append(block_entries[3])
                                        labeled_entries.append(block_entries[4])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                    return(labeled_entries)
                                
                                def parse_52A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":52A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)


                                def parse_52D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":52D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                
                                def parse_56A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":56A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def parse_56D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":56D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def parse_57A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":57A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"

                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                def parse_57D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":57D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                
                                def parse_58A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":58A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >=1  :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])

                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)


                                    return(labeled_entries)


                                def parse_58D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":58D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        #labeled_entries.append(block_entries[0] + block_entries[1]+ block_entries[2])
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                
                                def app_21(message_text):
                                    mt_message_21 = []

                                    try:

                                        if message_text is not None:
                                            mt_message_21.append(parse_first21_block(message_text))
                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_21

                                def app_01(message_text):
                                    mt_message_01 = []

                                    try:

                                        if message_text is not None:
                                            mt_message_01.append(parse_first01_block(message_text))
                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_01

                                def app_02_O(message_text):
                                    mt_message = []

                                    try:

                                        if message_text is not None:
                                            mt_message.append(parse_2nd_block_O(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message

                                def app_Set_Amount(message_text):
                                    mt_message_SA = []

                                    try:

                                        if message_text is not None:
                                            mt_message_SA.append(parse_Set_Amount(message_text))
                                        else : 
                                            mt_message_SA.append(parse_Set_Amount(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_SA
                                    
                                def app_52A(message_text):
                                    mt_message_52A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_52A.append(parse_52A(message_text))
                                        else : 
                                            mt_message_52A.append(parse_52A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_52A

                                def app_52D(message_text):
                                    mt_message_52D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_52D.append(parse_52D(message_text))
                                        else : 
                                            mt_message_52D.append(parse_52D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_52D
                                    
                                def app_56A(message_text):
                                    mt_message_56A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_56A.append(parse_56A(message_text))
                                        else : 
                                            mt_message_56A.append(parse_56A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_56A
                                def app_56D(message_text):
                                    mt_message_56D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_56D.append(parse_56D(message_text))
                                        else : 
                                            mt_message_56D.append(parse_56D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_56D
                                def app_57A(message_text):
                                    mt_message_57A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_57A.append(parse_57A(message_text))
                                        else : 
                                            mt_message_57A.append(parse_57A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_57A

                                def app_57D(message_text):
                                    mt_message_57D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_57D.append(parse_57D(message_text))
                                        else : 
                                            mt_message_57D.append(parse_57D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_57D

                                def app_58A(message_text):
                                    mt_message_58A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_58A.append(parse_58A(message_text))
                                        else : 
                                            mt_message_58A.append(parse_58A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_58A
                                
                                def app_58D(message_text):
                                    mt_message_58D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_58D.append(parse_58D(message_text))
                                        else : 
                                            mt_message_58D.append(parse_58D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_58D

                                w_res = app_21(ddd[op])
                                x_res = app_01(ddd[op])
                                y_res_I = app_02_O(ddd[op])

                                sa_res = app_Set_Amount(ddd[op])
                                
                                Ordering_A = app_52A(ddd[op])
                                Ordering_D = app_52D(ddd[op])
                                Intermediary_A = app_56A(ddd[op])
                                Intermediary_D = app_56D(ddd[op])
                                Account_A = app_57A(ddd[op])
                                Account_D = app_57D(ddd[op])

                                Beneficiary_A = app_58A(ddd[op])
                                Beneficiary_D = app_58D(ddd[op])

                                x_21 = pd.DataFrame.from_dict(w_res)
                                x_01 = pd.DataFrame.from_dict(x_res)
                                y_I = pd.DataFrame.from_dict(y_res_I)

                                sa = pd.DataFrame.from_dict(sa_res)
                                
                                O_A = pd.DataFrame.from_dict(Ordering_A)
                                O_D = pd.DataFrame.from_dict(Ordering_D)
                                I_A = pd.DataFrame.from_dict(Intermediary_A)
                                I_D = pd.DataFrame.from_dict(Intermediary_D)
                                A_A = pd.DataFrame.from_dict(Account_A)
                                A_D = pd.DataFrame.from_dict(Account_D)

                                B_A = pd.DataFrame.from_dict(Beneficiary_A)
                                B_D = pd.DataFrame.from_dict(Beneficiary_D)

                                x_21.columns = ['Receiver_BIC_F21','Session_Nbr_F21','Sequence_Nbr_F21']
                                x_01.columns = ['Receiver_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01']
                                y_I.columns = ['Sens_Swift','Msg_Type','In_time_date','Sender_BIC_CODE','Session','Isn','Out_time_date','Msg_Priority']

                                sa.columns = ['ValueDate', 'Currency', 'Amount']
                                
                                O_A.columns = ['52A_Party_Identifier','52A_Identifier_Code']
                                O_D.columns = ['52D_Party_Identifier','52D_Name_Address']
                                I_A.columns = ['56A_Party_Identifier','56A_Identifier_Code']
                                I_D.columns = ['56D_Party_Identifier','56D_Name_Address']
                                A_A.columns = ['57A_Party_Identifier','57A_Identifier_Code']
                                A_D.columns = ['57D_Party_Identifier','57D_Name_Address']

                                B_A.columns = ['58A_Party_Identifier','58A_Identifier_Code']
                                B_D.columns = ['58D_Party_Identifier','58D_Name_Address']
                                
                                R = pd.DataFrame.from_dict(x_01['Receiver_BIC_F01'])
                                R.columns = ['Receiver_BIC_Code']
                                

                                df = pd.read_sql_table('Banques_Correspondantes_Vostro', con=engine)
                                def search():
                                    s = 0
                                    for name in df.BIC:
                                        start = 0
                                        stop = len(y_I.Sender_BIC_CODE[0][:8])
                                        found = y_I.Sender_BIC_CODE[0][:8].find(name[:8], start, stop)
                                        if found == 0 : 
                                            s=1
                                    return s
                                search = search()

                                ss = pd.append([x_21,x_01, y_I, sa,O_A,O_D,I_A,I_D,A_A,A_D, B_A, B_D,R ], axis=1, join='inner')
                                k = [date] + [file_name] + [ddd[op]]
                                l = pd.DataFrame.from_dict(k)
                                m = l.T
                                m.columns = ['Treatment_Date','File_Name','Swift_Message']

                                u = pd.append([c, ss, m], axis=1, join='inner')

                                att =['Receiver_BIC_F21', 'Session_Nbr_F21', 'Sequence_Nbr_F21','Receiver_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01','Sens_Swift','Msg_Type','In_time_date','Sender_BIC_CODE','Etat','Session','Isn','Out_time_date','Msg_Priority','Block01_ACK','Text_bloc','Block01_FIN','Block02_Application_Header_input','Block02_Application_Header_output','Header_Block','UETR','20_Transaction_ref','21_Operation_code','13C_Time_indication','32A_Settled_amount','ValueDate', 'Currency', 'Amount','52A_Ordering_institution','52A_Party_Identifier','52A_Identifier_Code','52D_Ordering_institution','52D_Party_Identifier','52D_Name_Address','53_Sender_correspondent','54_Receiver_correspondent','56A_Intermediary','56A_Party_Identifier','56A_Identifier_Code','56D_Intermediary','56D_Party_Identifier','56D_Name_Address','57A_Account','57A_Party_Identifier','57A_Identifier_Code','57D_Account','57D_Party_Identifier','57D_Name_Address','58A_Beneficiary','58A_Party_Identifier','58A_Identifier_Code','58D_Beneficiary','58D_Party_Identifier','58D_Name_Address','72_Info','Trailer_Block','System_Trailer_Block','Treatment_Date','File_Name','Swift_Message']
                                all = pd.DataFrame(columns = att)

                                tab = all.append(u, ignore_index = True)
                                tab['Etat'] = np.where(search==1, 'A traiter','A ne pas traiter')

                                tab.to_sql(name='202',con=engine, index=False, if_exists='append')
                                logger.info(" Ligne ajoutée à la base de données avec succès")
                                logger.info(" Redirection du fichier " + "'" + fichier[i] + "'" + " vers " + MT202_traité)
                                logger.info(" Fin du traitement du fichier " + "'" + fichier[i] + "'")
                                
                            
                            except : 
                                #raise
                                pass
                            
                            op = op + 1
                        shutil.move(os.path.join(path,a),MT202_traité)
#_________________________________________________________INPUT______________________________________________________

            
            elif '{2:I202' in data :
                if ec > 1 :
                    shutil.move(os.path.join(path,a),MT202_non_traité)

                else : 
                    
                    if eh == 1 :
                        try : 

                            Decompose = data.replace("{1:F21", "F21").replace("}{1:F01", ",F01").replace("{1:F01", "F01").replace("}{2:I",",I").replace("{3:{108:",",{").replace("{3:{111:",",{").replace("{3:{121:",",{").replace("{121:",",{").replace("}}{4:", "}}").replace("}{4:{177:",",{177:").replace(':20:',',').replace(":21:", ",").replace(":13C:", ",").replace(":32A:",",").replace(":52A:",",").replace(":52D:",",").replace(":53A:",",").replace(":53B:",",").replace(":53C:",",").replace(":54A:",",").replace(":54B:",",").replace(":54D:",",").replace(":56A:",",").replace(":56D:",",").replace(":57A:",",").replace(":57B:",",").replace(":57D:",",").replace(":58A:",",").replace(":58D:",",").replace(":72:",",").replace("-}","").replace('{5:',',{').replace('{S:',',{').replace("-}","").replace("\x03","").split(',')

                            logger.info("\n" + 40*"-" + fichier[i]+ 40*"-" )
                            logger.info(" Décomposition du message ")

                            def DataFrame():
                                #Définition du DataFrame:
                                b = pd.DataFrame.from_dict(Decompose)
                                c = b.T

                                return(c)

                            def preparation():
                                #Détermination des colonnes:
                                column_names = []
                                column_names.clear()

                                L = str(Decompose)


                                if '{1:F01' in data :
                                    column_names.append('Block01_FIN')
                                if '{2:I202' in data :
                                    column_names.append('Block02_Application_Header_input')
                                if '{2:O202' in data :
                                    column_names.append('Block02_Application_Header_output')
                                if '{3:{108:' in data :
                                    column_names.append('Header_Block')
                                if '{3:{111:' in data :
                                    column_names.append('Header_Block')
                                if '{121' in data :
                                    column_names.append('UETR')

                                if ':20:' in data :
                                    column_names.append('20_Transaction_ref')

                                if ':21:' in data :
                                    column_names.append('21_Operation_code')


                                if ':13C:' in data : 
                                    column_names.append('13C_Time_indication')

                                if ':32A:' in data :
                                    column_names.append('32A_Settled_amount')

                                if ':52A:' in data :
                                    column_names.append('52A_Ordering_institution')
                                if ':52D:' in data :
                                    column_names.append('52D_Ordering_institution')

                                if ':53A:' in data :
                                    column_names.append('53_Sender_correspondent')
                                if ':53B:' in data :
                                    column_names.append('53_Sender_correspondent')
                                if ':53D:' in data :
                                    column_names.append('53_Sender_correspondent')

                                if ':54A:' in data :
                                    column_names.append('54_Receiver_correspondent')
                                if ':54B:' in data :
                                    column_names.append('54_Receiver_correspondent')
                                if ':54D:' in data :
                                    column_names.append('54_Receiver_correspondent')


                                if ':56A:' in data :
                                    column_names.append('56A_Intermediary')
                                if ':56D:' in data :
                                    column_names.append('56D_Intermediary')

                                if ':57A:'  in data :
                                    column_names.append('57A_Account')
                                if ':57B:' in data :
                                    column_names.append('57B_Account')
                                if ':57D:' in data :
                                    column_names.append('57D_Account')
                                    
                                    
                                if ':58A:' in data :
                                    column_names.append('58A_Beneficiary')
                                if ':58D:' in data :
                                    column_names.append('58D_Beneficiary')

                                if ':72:' in data :
                                    column_names.append('72_Info')


                                if '{5:' in data :
                                    column_names.append('Trailer_Block')
                                if '{S:' in data :
                                    column_names.append('System_Trailer_Block')
                                return(column_names)

                            c = DataFrame()
                            c.columns = preparation()

                            def parse_first21_block(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r"{1:F21(\w{12})(\d{4})(\d+)}"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 3:
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                    labeled_entries.append(block_entries[2])
                                return(labeled_entries)

                            def parse_first01_block(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r"{1:F01(\w{12})(\d{4})(\d+)}"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 3:
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                    labeled_entries.append(block_entries[2])
                                return(labeled_entries)

                            def parse_2nd_block_I(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = '{2:I(\d{3})(\w{12})(\w{1})(\d*)'
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))
                                if len(block_entries) >= 3:
                                    labeled_entries.append('Départ')
                                    labeled_entries.append( block_entries[0])
                                    labeled_entries.append( block_entries[1])
                                    if block_entries[2] == 'N' :
                                        labeled_entries.append('Normal')
                                    elif block_entries[2] =='S' :
                                        labeled_entries.append('System')
                                    elif block_entries[2] =='U' :
                                        labeled_entries.append('Urgent')

                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                return(labeled_entries)

                            def parse_Set_Amount(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":32A:(\d{2})(\d{2})(\d{2})(\w{3})(\d+\.\d*)"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) == 5 :
                                    labeled_entries.append( block_entries[2] + '-' + block_entries[1] + '-' + block_entries[0] )
                                    labeled_entries.append(block_entries[3])
                                    labeled_entries.append(block_entries[4])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)
                                return(labeled_entries)
                            
                            def parse_52A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":52A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)


                            def parse_52D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":52D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            
                            def parse_56A(msg):
                                block_entries = []
                                labeled_entries = []
                                
                                regex = r":56A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)

                            def parse_56D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":56D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            
                            def parse_57A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":57A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"

                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)
                            def parse_57D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":57D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)

                            def parse_58A(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":58A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >=1  :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])

                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)


                                return(labeled_entries)

                            
                            def parse_58D(msg):
                                block_entries = []
                                labeled_entries = []

                                regex = r":58D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                matches = re.finditer(regex, msg, re.MULTILINE)

                                for matchNum, match in enumerate(matches):
                                    matchNum += 1

                                    for groupNum in range(0, len(match.groups())):
                                        groupNum += 1
                                        block_entries.append(match.group(groupNum))

                                if len(block_entries) >= 1 :
                                    labeled_entries.append(block_entries[0])
                                    labeled_entries.append(block_entries[1])
                                if len(block_entries) == 0 :
                                    labeled_entries.append(np.nan)
                                    labeled_entries.append(np.nan)

                                return(labeled_entries)

                            def app_21(message_text):
                                mt_message_21 = []

                                try:

                                    if message_text is not None:
                                        mt_message_21.append(parse_first21_block(message_text))
                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_21

                            def app_01(message_text):
                                mt_message_01 = []

                                try:

                                    if message_text is not None:
                                        mt_message_01.append(parse_first01_block(message_text))
                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_01

                            def app_02_I(message_text):
                                mt_message = []

                                try:

                                    if message_text is not None:
                                        mt_message.append(parse_2nd_block_I(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message

                            def app_Set_Amount(message_text):
                                mt_message_SA = []

                                try:

                                    if message_text is not None:
                                        mt_message_SA.append(parse_Set_Amount(message_text))
                                    else : 
                                        mt_message_SA.append(parse_Set_Amount(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_SA
                                
                            def app_52A(message_text):
                                mt_message_52A = []

                                try:

                                    if message_text is not None:
                                        mt_message_52A.append(parse_52A(message_text))
                                    else : 
                                        mt_message_52A.append(parse_52A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_52A

                            def app_52D(message_text):
                                mt_message_52D = []

                                try:

                                    if message_text is not None:
                                        mt_message_52D.append(parse_52D(message_text))
                                    else : 
                                        mt_message_52D.append(parse_52D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_52D
                                
                            def app_56A(message_text):
                                mt_message_56A = []

                                try:

                                    if message_text is not None:
                                        mt_message_56A.append(parse_56A(message_text))
                                    else : 
                                        mt_message_56A.append(parse_56A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_56A
                            def app_56D(message_text):
                                mt_message_56D = []

                                try:

                                    if message_text is not None:
                                        mt_message_56D.append(parse_56D(message_text))
                                    else : 
                                        mt_message_56D.append(parse_56D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_56D
                            def app_57A(message_text):
                                mt_message_57A = []

                                try:

                                    if message_text is not None:
                                        mt_message_57A.append(parse_57A(message_text))
                                    else : 
                                        mt_message_57A.append(parse_57A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_57A
                                
                            def app_57D(message_text):
                                mt_message_57D = []

                                try:

                                    if message_text is not None:
                                        mt_message_57D.append(parse_57D(message_text))
                                    else : 
                                        mt_message_57D.append(parse_57D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_57D

                            def app_58A(message_text):
                                mt_message_58A = []

                                try:

                                    if message_text is not None:
                                        mt_message_58A.append(parse_58A(message_text))
                                    else : 
                                        mt_message_58A.append(parse_58A(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_58A
                            
                            def app_58D(message_text):
                                mt_message_58D = []

                                try:

                                    if message_text is not None:
                                        mt_message_58D.append(parse_58D(message_text))
                                    else : 
                                        mt_message_58D.append(parse_58D(message_text))

                                except Exception as ex:
                                    print('Exception in parse')
                                    print(str(ex))
                                finally:
                                    return mt_message_58D

                            x_res = app_01(data)
                            y_res_I = app_02_I(data)

                            sa_res = app_Set_Amount(data)
                            
                            Ordering_A = app_52A(data)
                            Ordering_D = app_52D(data)
                            Intermediary_A = app_56A(data)
                            Intermediary_D = app_56D(data)
                            Account_A = app_57A(data)
                            Account_D = app_57D(data)

                            Beneficiary_A = app_58A(data)
                            Beneficiary_D = app_58D(data)

                            x_01 = pd.DataFrame.from_dict(x_res)
                            y_I = pd.DataFrame.from_dict(y_res_I)

                            sa = pd.DataFrame.from_dict(sa_res)
                            
                            O_A = pd.DataFrame.from_dict(Ordering_A)
                            O_D = pd.DataFrame.from_dict(Ordering_D)
                            I_A = pd.DataFrame.from_dict(Intermediary_A)
                            I_D = pd.DataFrame.from_dict(Intermediary_D)
                            A_A = pd.DataFrame.from_dict(Account_A)
                            A_D = pd.DataFrame.from_dict(Account_D)

                            B_A = pd.DataFrame.from_dict(Beneficiary_A)
                            B_D = pd.DataFrame.from_dict(Beneficiary_D)

                            x_01.columns = ['Sender_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01']
                            y_I.columns = ['Sens_Swift','Msg_Type','Receiver_BIC_CODE','Msg_Priority']

                            sa.columns = ['ValueDate', 'Currency', 'Amount']
                            
                            O_A.columns = ['52A_Party_Identifier','52A_Identifier_Code']
                            O_D.columns = ['52D_Party_Identifier','52D_Name_Address']
                            I_A.columns = ['56A_Party_Identifier','56A_Identifier_Code']
                            I_D.columns = ['56D_Party_Identifier','56D_Name_Address']
                            A_A.columns = ['57A_Party_Identifier','57A_Identifier_Code']
                            A_D.columns = ['57D_Party_Identifier','57D_Name_Address']

                            B_A.columns = ['58A_Party_Identifier','58A_Identifier_Code']
                            B_D.columns = ['58D_Party_Identifier','58D_Name_Address']
                            
                            S = pd.DataFrame.from_dict(x_01['Sender_BIC_F01'])
                            S.columns = ['Sender_BIC_Code']

                            ss = pd.append([ x_01, y_I, sa,O_A,O_D,I_A,I_D,A_A,A_D, B_A,B_D,S ], axis=1, join='inner')
                            k = [date] + [file_name] + [data]
                            l = pd.DataFrame.from_dict(k)
                            m = l.T

                            m.columns = ['Treatment_Date','File_Name','Swift_Message']
                            u = pd.append([c, ss, m], axis=1, join='inner')

                            att =['Receiver_BIC_F21', 'Session_Nbr_F21', 'Sequence_Nbr_F21','Sender_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01','Sens_Swift','Msg_Type','In_time_date','Receiver_BIC_CODE','Session','Isn','Out_time_date','Msg_Priority','Block01_ACK','Text_bloc','Block01_FIN','Block02_Application_Header_input','Block02_Application_Header_output','Header_Block','UETR','20_Transaction_ref','21_Operation_code','13C_Time_indication','32A_Settled_amount','ValueDate', 'Currency', 'Amount','52A_Ordering_institution','52A_Party_Identifier','52A_Identifier_Code','52D_Ordering_institution','52D_Party_Identifier','52D_Name_Address','53_Sender_correspondent','54_Receiver_correspondent','56A_Intermediary','56A_Party_Identifier','56A_Identifier_Code','56D_Intermediary','56D_Party_Identifier','56D_Name_Address','57A_Account','57A_Party_Identifier','57A_Identifier_Code','57D_Account','57D_Party_Identifier','57D_Name_Address','58A_Beneficiary','58A_Party_Identifier','58A_Identifier_Code','58D_Beneficiary','58D_Party_Identifier','58D_Name_Address','72_Info','Trailer_Block','System_Trailer_Block','Treatment_Date','File_Name','Swift_Message']

                            all = pd.DataFrame(columns = att)

                            tab = all.append(u, ignore_index = True)

                            tab.to_sql(name='202',con=engine, index=False, if_exists='append')

                            shutil.move(os.path.join(path,a),MT202_traité)

                            logger.info(" Ligne ajoutée à la base de données avec succès")
                            logger.info(" Redirection du fichier " + "'" + fichier[i] + "'" + " vers " + MT202_traité)
                            logger.info(" Fin du traitement du fichier " + "'" + fichier[i] + "'")
                        
                        except : 
                            #raise
                            pass 
#___________________________________________INPUT_Rép____________________________________________________________

                    elif eh > 1 :
                        
                        ddd = data.split('\x03')
                        #lgr = len(ddd)
                        lgr = len(ddd) - 1
                        op = 0
                        
                        for op in range(lgr):
                            try : 
                                       
                                Decompose = ddd[op].replace("{1:F21", "F21").replace("}{1:F01", ",F01").replace("{1:F01", "F01").replace("}{2:I",",I").replace("{3:{108:",",{").replace("{3:{111:",",{").replace("{121:",",{").replace("}}{4:", "}}").replace("}{4:{177:",",{177:").replace(':20:',',').replace(":21:", ",").replace(":13C:", ",").replace(":32A:",",").replace(":52A:",",").replace(":52D:",",").replace(":53A:",",").replace(":53B:",",").replace(":53C:",",").replace(":54A:",",").replace(":54B:",",").replace(":54D:",",").replace(":56A:",",").replace(":56D:",",").replace(":57A:",",").replace(":57B:",",").replace(":57D:",",").replace(":58A:",",").replace(":58D:",",").replace(":72:",",").replace("-}","").replace('{5:',',{').replace('{S:',',{').replace("-}","").split(',')

                                logger.info("\n" + 40*"-" + fichier[i]+ 40*"-" )
                                logger.info(" Décomposition du message N°" + str(op))
                                #if '{2:I103' or '{2:O103' in ddd[op]:
                                if not '{2:I202' in ddd[op]:
                                    logger.info(" Redirection du message vers " + path + " sous le nom de " + file_name + '_' + str(op) + '.txt')
                                    with open(path + '/' + file_name + '_' + str(op) + '.txt', 'x') as f:
                                        f.write(ddd[op])
                                    #shutil.copy(os.path.join(path,a),MT202_non_traité)

                                def DataFrame():
                                    #Définition du DataFrame:
                                    b = pd.DataFrame.from_dict(Decompose)
                                    c = b.T

                                    return(c)

                                def preparation():
                                    #Détermination des colonnes:
                                    L = str(Decompose)

                                    column_names = []
                                    column_names.clear()


                                    if '{1:F01' in ddd[op] :
                                        column_names.append('Block01_FIN')
                                    if '{2:I202' in ddd[op] :
                                        column_names.append('Block02_Application_Header_input')
                                    if '{2:O202' in ddd[op] :
                                        column_names.append('Block02_Application_Header_output')
                                    if '{3:{108:' in ddd[op] :
                                        column_names.append('Header_Block')
                                    if '{3:{111:' in ddd[op] :
                                        column_names.append('Header_Block')
                                    if '{121' in ddd[op] :
                                        column_names.append('UETR')

                                    if ':20:' in ddd[op] :
                                        column_names.append('20_Transaction_ref')

                                    if ':21:' in ddd[op] :
                                        column_names.append('21_Operation_code')

                                    if ':13C:' in ddd[op] : 
                                        column_names.append('13C_Time_indication')

                                    if ':32A:' in ddd[op] :
                                        column_names.append('32A_Settled_amount')

                                    if ':52A:' in ddd[op] :
                                        column_names.append('52A_Ordering_institution')
                                    if ':52D:' in ddd[op] :
                                        column_names.append('52D_Ordering_institution')

                                    if ':53A:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')
                                    if ':53B:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')
                                    if ':53D:' in ddd[op] :
                                        column_names.append('53_Sender_correspondent')

                                    if ':54A:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')
                                    if ':54B:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')
                                    if ':54D:' in ddd[op] :
                                        column_names.append('54_Receiver_correspondent')

                                    if ':56A:' in ddd[op] :
                                        column_names.append('56A_Intermediary')
                                    if ':56D:' in ddd[op] :
                                        column_names.append('56D_Intermediary')

                                    if ':57A:'  in ddd[op] :
                                        column_names.append('57A_Account')
                                    if ':57B:' in ddd[op] :
                                        column_names.append('57B_Account')
                                    if ':57D:' in ddd[op] :
                                        column_names.append('57D_Account')


                                    if ':58A:' in ddd[op] :
                                        column_names.append('58A_Beneficiary')
                                    if ':58D:' in ddd[op] :
                                        column_names.append('58D_Beneficiary')

                                    if ':72:' in ddd[op] :
                                        column_names.append('72_Info')

                                    if '{5:' in ddd[op] :
                                        column_names.append('Trailer_Block')
                                    if '{S:' in ddd[op] :
                                        column_names.append('System_Trailer_Block')

                                    return (column_names)

                                c = DataFrame()
                                c.columns = preparation()

                                def parse_first21_block(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r"{1:F21(\w{12})(\d{4})(\d+)}"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 3:
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                        labeled_entries.append(block_entries[2])
                                    return(labeled_entries)

                                def parse_first01_block(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r"{1:F01(\w{12})(\d{4})(\d+)}"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 3:
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                        labeled_entries.append(block_entries[2])
                                    return(labeled_entries)

                                def parse_2nd_block_I(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r'{2:I(\d{3})(\w{12})(\w{1})(\d*)'
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))
                                    if len(block_entries) >= 3:
                                        labeled_entries.append('Départ')
                                        labeled_entries.append( block_entries[0])
                                        labeled_entries.append( block_entries[1])
                                        if block_entries[2] == 'N' :
                                            labeled_entries.append('Normal')
                                        elif block_entries[2] =='S' :
                                            labeled_entries.append('System')
                                        elif block_entries[2] =='U' :
                                            labeled_entries.append('Urgent')

                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                    return(labeled_entries)

                                def parse_Set_Amount(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":32A:(\d{2})(\d{2})(\d{2})(\w{3})(\d+\.\d*)"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) == 5 :
                                        labeled_entries.append( block_entries[2] + '-' + block_entries[1] + '-' + block_entries[0] )
                                        labeled_entries.append(block_entries[3])
                                        labeled_entries.append(block_entries[4])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                    return(labeled_entries)
                                
                                def parse_52A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":52A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)


                                def parse_52D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":52D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        #labeled_entries.append(block_entries[0] + block_entries[1]+ block_entries[2])
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        #labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                
                                def parse_56A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":56A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def parse_56D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":56D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def parse_57A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":57A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"

                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)
                                def parse_57D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":57D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def parse_58A(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":58A:([/]+[A-Z0-9]{0,35}[0-9]+\s*)*([A-Z]{6}[A-Z0-9]*[A-Z]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >=1  :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])

                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)


                                    return(labeled_entries)
                                
                                def parse_58D(msg):
                                    block_entries = []
                                    labeled_entries = []

                                    regex = r":58D:([/]+[A-Z0-9\S]{0,35})*([A-Z0-9 \t\n\r\f\v]+)*"
                                    matches = re.finditer(regex, msg, re.MULTILINE)

                                    for matchNum, match in enumerate(matches):
                                        matchNum += 1

                                        for groupNum in range(0, len(match.groups())):
                                            groupNum += 1
                                            block_entries.append(match.group(groupNum))

                                    if len(block_entries) >= 1 :
                                        labeled_entries.append(block_entries[0])
                                        labeled_entries.append(block_entries[1])
                                    if len(block_entries) == 0 :
                                        labeled_entries.append(np.nan)
                                        labeled_entries.append(np.nan)

                                    return(labeled_entries)

                                def app_21(message_text):
                                    mt_message_21 = []

                                    try:

                                        if message_text is not None:
                                            mt_message_21.append(parse_first21_block(message_text))
                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_21

                                def app_01(message_text):
                                    mt_message_01 = []

                                    try:

                                        if message_text is not None:
                                            mt_message_01.append(parse_first01_block(message_text))
                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_01

                                def app_02_I(message_text):
                                    mt_message = []

                                    try:

                                        if message_text is not None:
                                            mt_message.append(parse_2nd_block_I(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message

                                def app_Set_Amount(message_text):
                                    mt_message_SA = []

                                    try:

                                        if message_text is not None:
                                            mt_message_SA.append(parse_Set_Amount(message_text))
                                        else : 
                                            mt_message_SA.append(parse_Set_Amount(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_SA
                                def app_52A(message_text):
                                    mt_message_52A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_52A.append(parse_52A(message_text))
                                        else : 
                                            mt_message_52A.append(parse_52A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_52A

                                def app_52D(message_text):
                                    mt_message_52D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_52D.append(parse_52D(message_text))
                                        else : 
                                            mt_message_52D.append(parse_52D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_52D
                                    
                                def app_56A(message_text):
                                    mt_message_56A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_56A.append(parse_56A(message_text))
                                        else : 
                                            mt_message_56A.append(parse_56A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_56A
                                def app_56D(message_text):
                                    mt_message_56D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_56D.append(parse_56D(message_text))
                                        else : 
                                            mt_message_56D.append(parse_56D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_56D
                                def app_57A(message_text):
                                    mt_message_57A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_57A.append(parse_57A(message_text))
                                        else : 
                                            mt_message_57A.append(parse_57A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_57A

                                def app_57D(message_text):
                                    mt_message_57D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_57D.append(parse_57D(message_text))
                                        else : 
                                            mt_message_57D.append(parse_57D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_57D

                                def app_58A(message_text):
                                    mt_message_58A = []

                                    try:

                                        if message_text is not None:
                                            mt_message_58A.append(parse_58A(message_text))
                                        else : 
                                            mt_message_58A.append(parse_58A(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_58A
                                
                                def app_58D(message_text):
                                    mt_message_58D = []

                                    try:

                                        if message_text is not None:
                                            mt_message_58D.append(parse_58D(message_text))
                                        else : 
                                            mt_message_58D.append(parse_58D(message_text))

                                    except Exception as ex:
                                        print('Exception in parse')
                                        print(str(ex))
                                    finally:
                                        return mt_message_58D

                                x_res = app_01(ddd[op])
                                y_res_I = app_02_I(ddd[op])

                                sa_res = app_Set_Amount(ddd[op])
                                
                                Ordering_A = app_52A(ddd[op])
                                Ordering_D = app_52D(ddd[op])
                                Intermediary_A = app_56A(ddd[op])
                                Intermediary_D = app_56D(ddd[op])
                                Account_A = app_57A(ddd[op])
                                Account_D = app_57D(ddd[op])

                                Beneficiary_A = app_58A(ddd[op])
                                Beneficiary_D = app_58D(ddd[op])
                                
                                x_01 = pd.DataFrame.from_dict(x_res)
                                y_I = pd.DataFrame.from_dict(y_res_I)

                                sa = pd.DataFrame.from_dict(sa_res)
                                
                                O_A = pd.DataFrame.from_dict(Ordering_A)
                                O_D = pd.DataFrame.from_dict(Ordering_D)
                                I_A = pd.DataFrame.from_dict(Intermediary_A)
                                I_D = pd.DataFrame.from_dict(Intermediary_D)
                                A_A = pd.DataFrame.from_dict(Account_A)
                                A_D = pd.DataFrame.from_dict(Account_D)

                                B_A = pd.DataFrame.from_dict(Beneficiary_A)
                                B_D = pd.DataFrame.from_dict(Beneficiary_D)

                                x_01.columns = ['Sender_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01']
                                y_I.columns = ['Sens_Swift','Msg_Type','Receiver_BIC_CODE','Msg_Priority']

                                sa.columns = ['ValueDate', 'Currency', 'Amount']
                                
                                O_A.columns = ['52A_Party_Identifier','52A_Identifier_Code']
                                O_D.columns = ['52D_Party_Identifier','52D_Name_Address']
                                I_A.columns = ['56A_Party_Identifier','56A_Identifier_Code']
                                I_D.columns = ['56D_Party_Identifier','56D_Name_Address']
                                A_A.columns = ['57A_Party_Identifier','57A_Identifier_Code']
                                A_D.columns = ['57D_Party_Identifier','57D_Name_Address']

                                B_A.columns = ['58A_Party_Identifier','58A_Identifier_Code']
                                B_D.columns = ['58D_Party_Identifier','58D_Name_Address']
                                
                                S = pd.DataFrame.from_dict(x_01['Sender_BIC_F01'])
                                S.columns = ['Sender_BIC_Code']
                                
                                ss = pd.append([x_01, y_I, sa,O_A,O_D,I_A,I_D,A_A,A_D,B_A,B_D,S ], axis=1, join='inner')
                                k = [date] + [file_name] + [ddd[op]]
                                l = pd.DataFrame.from_dict(k)
                                m = l.T

                                m.columns = ['Treatment_Date','File_Name','Swift_Message']
                                u = pd.append([c, ss, m], axis=1, join='inner')

                                att =['Receiver_BIC_F21', 'Session_Nbr_F21', 'Sequence_Nbr_F21','Sender_BIC_F01','Session_Nbr_F01','Sequence_Nbr_F01','Sens_Swift','Msg_Type','In_time_date','Receiver_BIC_CODE','Session','Isn','Out_time_date','Msg_Priority','Block01_ACK','Text_bloc','Block01_FIN','Block02_Application_Header_input','Block02_Application_Header_output','Header_Block','UETR','20_Transaction_ref','21_Operation_code','13C_Time_indication','32A_Settled_amount','ValueDate', 'Currency', 'Amount','52A_Ordering_institution','52A_Party_Identifier','52A_Identifier_Code','52D_Ordering_institution','52D_Party_Identifier','52D_Name_Address','53_Sender_correspondent','54_Receiver_correspondent','56A_Intermediary','56A_Party_Identifier','56A_Identifier_Code','56D_Intermediary','56D_Party_Identifier','56D_Name_Address','57A_Account','57A_Party_Identifier','57A_Identifier_Code','57D_Account','57D_Party_Identifier','57D_Name_Address','58A_Beneficiary','58A_Party_Identifier','58A_Identifier_Code','58D_Beneficiary','58D_Party_Identifier','58D_Name_Address','72_Info','Trailer_Block','System_Trailer_Block','Treatment_Date','File_Name','Swift_Message']

                                all = pd.DataFrame(columns = att)

                                tab = all.append(u, ignore_index = True)

                                tab.to_sql(name='202',con=engine, index=False, if_exists='append')

                                logger.info(" Ligne ajoutée à la base de données avec succès")
                                logger.info(" Redirection du fichier " + "'" + fichier[i] + "'" + " vers " + MT202_traité)
                                logger.info(" Fin du traitement du fichier " + "'" + fichier[i] + "'")

                            except :
                                #raise
                                pass
                            
                            op = op + 1    
                        shutil.move(os.path.join(path,a),MT202_traité)
                        
                        
            else :
                pass
    
    i = i + 1
    logger.info(" \n ")
    logger.info(" Fin du traitement ")
    logger.info(140*"_")
    logger.info(" \n ")
