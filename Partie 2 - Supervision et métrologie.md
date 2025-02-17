# 3. Partie 2 - Supervision et métrologie avec SNMP
Objectifs
- Mise en place d'un agent SNMPv3 et SNMPv2
- Sécurisation des trames SNMP
- Etude de la métrologie via SNMP

**************************************************
# 4.1 Configuration de SNMPv3 dans les routeurs
Après avoir configuré SNMPv3 sur les routeurs, nous avons installé les outils snmp sur les machines A et B afin de réaliser des tests.
Nous avons par la suite testé de récupérer l'hostname du routeur 813-B10-R1 via la commande ci-dessous. Nous avons fait la même chose pour tester la configuration de 813-B10-R2
```
[etudiant@813-B10-B ~]$ snmpget -v3 -u snmpuser -l authPriv -a SHA -A auth_pass -x AES -X crypt_pass 10.200.2.254 sysName.0
```
Le résultat est correct : ``` SNMPv2-MIB::sysName.0 = STRING: 813-B10-R1 ```

### <u> Question 8 </u>
Afin de récupérer l'objet "syslocation" nous utilisons la commande snmpget suivante :
```

```
**************************************************
# 4.2 Configuration de SNMPv2 dans les routeurs
### <u> Question 9 </u>



### <u> Question 10 </u>



### <u> Question 11 </u>



### <u> Question 12 </u>



### <u> Question 13 </u>



**************************************************
# 4.3 Métrologie
### <u> Question 15 </u>



### <u> Question 16 </u>



### <u> Question 17 </u>

