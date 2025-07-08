#!/bin/bash
 
# Remove old files
rm -rf /tmp/Default_conn.txt
 
# Get pod names
n0=$(kubectl get pods --namespace=default | grep pgpool | grep Running | awk '{ print $1 }')
n1=$(kubectl get pods -n am-finrep | grep pgpool | grep Running | awk '{ print $1 }')
 
# Check if pods were found
if [ -z "$n0" ] || [ -z "$n1" ]; then
    echo "Error: Could not find running pgpool pods in one or both namespaces"
    exit 1
fi
 
# Get connections counts
T_CON_0=$(kubectl exec -it --namespace=default $n0 --container pgpool -- sh -c "ps -ef | grep pgpool | wc -l" | tr -dc '0-9')
A_CON_0=$(kubectl exec -it --namespace=default $n0 --container pgpool -- sh -c "ps -ef | grep pgpool | grep wait | wc -l" | tr -dc '0-9')
T_CON_1=$(kubectl exec -it --namespace=am-finrep $n1 --container pgpool -- sh -c "ps -ef | grep pgpool | wc -l" | tr -dc '0-9')
A_CON_1=$(kubectl exec -it --namespace=am-finrep $n1 --container pgpool -- sh -c "ps -ef | grep pgpool | grep wait | wc -l" | tr -dc '0-9')
 
# Get service connections using the provided command
service_conn_default=$(kubectl exec -it --namespace=default $n0 --container pgpool -- sh -c "ps -ef | grep -i 'pgpool' | grep -v 'grep\|-n\|health\|PCP\|worker' | awk '{ print \$9 }' | awk -F, 'NR>1{arr[\$1]++}END{for (a in arr) print arr[a], a}' | sort -n -r | awk '{ print \$1\"#\"\$2 }'")
service_conn_finrep=$(kubectl exec -it --namespace=am-finrep $n1 --container pgpool -- sh -c "ps -ef | grep -i 'pgpool' | grep -v 'grep\|-n\|health\|PCP\|worker' | awk '{ print \$9 }' | awk -F, 'NR>1{arr[\$1]++}END{for (a in arr) print arr[a], a}' | sort -n -r | awk '{ print \$1\"#\"\$2 }'")
 
{
    # Print summary table
    echo "+--------------------+---------------------+---------------------+"
    echo "| Namespace          | Total Connections   | Available           |"
    echo "+--------------------+---------------------+---------------------+"
    printf "| %-18s | %19s | %19s |\n" "default" "$T_CON_0" "$A_CON_0"
    printf "| %-18s | %19s | %19s |\n" "am-finrep" "$T_CON_1" "$A_CON_1"
    echo "+--------------------+---------------------+---------------------+"
    # Print default namespace service connections
    echo -e "\nDefault Namespace Connection Details"
    echo "Count#ServiceName"
    echo "$service_conn_default"
    # Print am-finrep namespace service connections
    echo -e "\nam-finrep Namespace Connection Details"
    echo "Count#ServiceName"
    echo "$service_conn_finrep"
} | tee /tmp/Default_conn.txt
