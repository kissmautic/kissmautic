# Pull & Play, extremely verbose, KISS Dockerfile for Mautic 5.1.1

...
   _____                  __  __             _   _        ____  _  ______  
  | ____|__ _ ___ _   _  |  \/  | __ _ _   _| |_(_) ___  |  _ \| |/ /  _ \ 
  |  _| / _` / __| | | | | |\/| |/ _` | | | | __| |/ __| | | | | ' /| |_) |
  | |__| (_| \__ \ |_| | | |  | | (_| | |_| | |_| | (__  | |_| | . \|  _ < 
  |_____\__,_|___/\__, | |_|  |_|\__,_|\__,_|\__|_|\___| |____/|_|\_\_| \_\
                   |___/                                                    
...

 Intended uses: 
 - A simple way to test Mautic.
 - An live overview of Mautic requirements.
 - A starting point for Mautic code exploration and tinkering.
 It is NOT meant as a production ready image.

 If you have any ideas how to make it clearer or simpler, please submit a PR or an Issue.
 Otherwise, please consider one of these other repos: 
 kissmautic/mautic-advanced, For production deployments.
 kissmautic/mautic-dev, For development stacks.

#
#                                 --                                   
#                           ((((((()))))))                              
#                      ((((((((         ))))))                          
#                   (((((                      ,                     
#                ((((                        ***    )))                  
#              ((((                        ****      )))           
#             ((((          ,             ****        )))                 
#            (((           ****         ****           )))              
#           (((           *******     *****             )))             
#           (((          ********** ******  **          )))             
#           (((         ****   *********   ****         )))             
#           (((        ****     ******      ****        )))             
#           (((       ****         *         ****       )))             
#            (((     ****                     ****     )))              
#             ((((                                   ))))               
#               (((                                 )))                 
#                (((((                           )))))                  
#                   (((((                     )))))                     
#                       (((((((         )))))))                        
#                            (((((()))))))                              
#                                 --                                   
#                      _     _                _                   
#                     | |   | |              | |                  
#           ________  | | __| |_  ____     __| |  ___ __   __     
#          |  _   _ \ | |/ /| __|/ _  |   / _  | / _ \  \ / /           
#          | | | | | ||   < | |_| (_| | _| (_| ||  __/ \ V /      
#          |_| |_| |_||_|\_\ \__|\___ |(_)\____| \___|  \_/       
#                         _       __/ |                           
#                        | |     |___/                            
#   ________   ____ _   _| |_ ___  ____ _______        ___  ____ ____ 
#  |  _   _ \ / _  | | | | __/ _ \/ _  |  _   _ \     / _ \|  __/ _  |
#  | | | | | | (_| | |_| | ||  __/ (_| | | | | | | _  |(_) | | | (_| |
#  |_| |_| |_|\__,_|\__,_|\__\___|\__,_|_| |_| |_|(_) \___/|_|  \__, |
#                                                                __/ |
#                                                               |___/ 
#                                                                         

