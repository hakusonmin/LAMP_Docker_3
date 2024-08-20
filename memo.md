portはenvを見てください

起動スクリプトは  
docker compose -p project-name up 
今回の場合は docker compose -p proto_project_lamp_drupal up  

継承スタイルのcompose.yamlよりホスト側のIPをenvで制御できるようにした方がいいのでは？