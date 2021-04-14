# getHashCodeFromInvoice
This is a script that retrieves all invoice data from the Sap B1 database and validates the hash code.


SELECT DocEntry
        , DocDate
        , Serial
        , SeriesStr
        , State
        , CityS
        , REPLACE(CAST( DocTotal  AS NUMERIC(19,2)),'.',',') as DocTotal
        , NfmName
        , U_numeroExtrato
        , REPLACE(CAST( DrawnSum  AS NUMERIC(19,2)),'.',',') as DrawnSum
        , HASHCALCULADO
        , HASHSAP 
        FROM (
            SELECT T0.DocEntry,
            T0.DocDate, 
            T0.Serial, 
            T0.SeriesStr,
            T5.State, 
            T5.CityS, 
            T0.DocTotal, 
            T0.U_numeroExtrato, 
            T6.NfmName, 
            INV9.DrawnSum
            , SUBSTRING(
                UPPER(
                    sys.fn_sqlvarbasetostr(
                        HASHBYTES('MD5',
                            CONCAT( REPLICATE('0' , (14 - LEN (REPLACE(REPLACE(REPLACE(OCRD.Password, '.', ''), '/', ''), '-', '')))) 
                                + CAST (REPLACE(REPLACE(REPLACE(OCRD.Password, '.', ''), '/', ''), '-', '') AS VARCHAR(14))
                            , REPLICATE('0' , (9 - LEN  (T0.Serial))) 
                            	+ CAST (T0.Serial AS VARCHAR(9))
                            , REPLICATE('0' , (12 - LEN (REPLACE(REPLACE(FORMAT(COALESCE(INV9.DrawnSum, 0) + T0.DocTotal, 'F', 'en-us'), '.', ''),'.', ',')))) 
                                + CAST (REPLACE(REPLACE(FORMAT(COALESCE(INV9.DrawnSum, 0) + T0.DocTotal, 'F', 'en-us'), '.', ''),'.', ',') AS VARCHAR(12))
                            , REPLICATE('0' , (12 - LEN (REPLACE(FORMAT(SUM(T2.BaseSum), 'F', 'en-us'), '.', '')))) 
                                + CAST (REPLACE(FORMAT(SUM(T2.BaseSum), 'F', 'en-us'), '.', '') AS VARCHAR(12))
                            , REPLICATE('0' , (12 - LEN (REPLACE(FORMAT(SUM(case when T2.StaCode like '%ICMS%' then T2.TaxSum else 0 end), 'F', 'en-us'), '.', '')))) 
                                + CAST (REPLACE(FORMAT(SUM(case when T2.StaCode like '%ICMS%' then T2.TaxSum else 0 end), 'F', 'en-us'), '.', '') AS VARCHAR(12))
                            , CONVERT(VARCHAR(8), T0.DocDate, 112)
                            , REPLICATE('0' , (14 - LEN (REPLACE(REPLACE(REPLACE(T0.VATRegNum, '.', ''), '/', ''), '-', '')))) 
                                + CAST (REPLACE(REPLACE(REPLACE(T0.VATRegNum, '.', ''), '/', ''), '-', '') AS VARCHAR(14))
                            )
                        )
                    )
                ),3,32) as HASHCALCULADO
            , REPLACE(T0.U_ChAutent, '.', '') as HASHSAP
            FROM OINV T0
            INNER JOIN INV1 T1 ON T1.DocEntry = t0.DocEntry
            INNER JOIN INV4 T2 ON T2.DocEntry = t1.DocEntry AND t2.LineNum = t1.LineNum
            INNER JOIN OCRD ON T0.CardCode = OCRD.CardCode
            INNER JOIN INV12 T5 ON T0.DocEntry = T5.DocEntry
            INNER JOIN ONFM T6 ON T6.AbsEntry = T0.Model
            LEFT JOIN OSTT T3 ON T3.AbsId = T2.staType
            LEFT JOIN ONFT T4 ON T4.AbsId = T3.NfTaxId                                        
            LEFT JOIN INV9 ON T0.DocEntry = INV9.DocEntry                                      
            WHERE T4.AbsId in (-6, -3)
            AND T0.Model in (18, 19)
            AND T0.DocDate >= DATEADD(day,-7, GETDATE())
            AND T0.Canceled = 'N'
            GROUP BY
            T0.DocEntry, 
            OCRD.Password, 
            T0.Serial, 
            T0.DocTotal,
            T0.U_ChAutent,
            T0.VATRegNum, 
            T0.DocDate, 
            T0.SeriesStr, 
            T5.State, 
            T5.CityS,
            T0.DocTotal,
            T0.U_numeroExtrato, 
            T6.NfmName,
            INV9.DrawnSum
        ) as t
        WHERE t.HASHCALCULADO <> t.HASHSAP
