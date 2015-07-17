     LoadLeveler and PE require that userids be common across the cluster. There are many tools and services available to manage userids and passwords across large numbers of nodes. One simple way is to use common /etc/password files across your cluster. You can do this using xCAT's syncfiles function. Create the following file: 
    
      vi /install/custom/install/aix/&lt;profile&gt;.synclist
     add the following lines:
       /etc/hosts /etc/passwd /etc/group -&gt; /etc/
       /etc/security/passwd /etc/security/group /etc/security/limits /etc/security/roles -&gt; /etc/security/
    

     Add this syncfile to your image: 
    
      chdef -t osimage -o &lt;imagename&gt; synclists=/install/custom/install/aix/&lt;profile&gt;.synclist
    

    When the node is installed or 'updatenode &lt;noderange&gt; -F' is run, these files will be copied to the node. You can periodically re-sync these files as changes occur in your cluster. See the xCAT documentation for more details: [Sync-ing_Config_Files_to_Nodes]. 