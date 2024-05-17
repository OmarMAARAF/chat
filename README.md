hey i have this eneties :
@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class IncidentSLA {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private LocalDateTime startDate;
    private LocalDateTime dueDate;
    private LocalDateTime closingDate;
    private String status;
    @ManyToOne (fetch = FetchType.LAZY)
    @JoinColumn(name="incident_id")
    @JsonIgnore
    private Incident incident;
    @ManyToOne
    private TypeIncidentSLA typeIncidentSLA;
    @Column(updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
public class SLA {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;
    private String code;
    private String startStatus;
    private String endStatus;
    @Column(columnDefinition = "BOOLEAN DEFAULT true")
    private boolean enabled;
    @Column(updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    @UpdateTimestamp
    private LocalDateTime updatedAt;

}

@Entity
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Builder
public class TypeIncidentSLA {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @ManyToOne
    @JoinColumn(name = "type_id")
    @JsonIgnore
    private TypeIncident typeIncident;

    @ManyToOne
    @JoinColumn(name = "sla_id")
    private SLA sla;

    private float duration;
    @Column(updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}

public class Incident {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titre;

    @ManyToOne
    @JoinColumn(name = "reporter_id")
    private User reporter;

    @ManyToOne
    @JoinColumn(name = "assigne_id")
    private User assigne;

    @ManyToOne
    @JoinColumn(name = "family_id")
    private FamilyIncident familyIncident;

    @ManyToOne
    @JoinColumn(name = "type_id")
    private TypeIncident typeIncident;

    @ManyToOne
    @JoinColumn(name = "entitie_id")
    private Entitie entitie;
    private String processInstanceId;
    private String currentActivityId;
    private String currentActivityName;
    @OneToMany(mappedBy = "incident",fetch = FetchType.LAZY)
    private Set<AttachementIncident> attachementIncidents;
    private String description;
    private String numeroTelephone;
    private boolean enabled;
    @Enumerated(EnumType.STRING)
    private IncidentStatus status;
    @Column(name = "ciriticité")
    private String severity;
    @Column(updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    private LocalDateTime endOfTreatmentAt;
    private Long attachement;
    @ManyToOne
    @JoinColumn(name="request_id",referencedColumnName = "id")
    private IncidentRequest request;
    @OneToMany(mappedBy = "incident",fetch = FetchType.LAZY)
    private List<Comment> comments =new ArrayList<>();
    @OneToMany(mappedBy="incident",fetch = FetchType.LAZY)
    private List<IncidentHistory> incidentHistories=new ArrayList<>();
    @OneToMany(mappedBy = "incident")
    private List<IncidentAction> incidentActions=new ArrayList<>();
    @ManyToOne
    private Agence agence;
    @ManyToOne()
    private MotifRejet rejectReason;
    private String rejectDescription;
    @OneToMany (mappedBy = "incident" ,fetch = FetchType.LAZY)
    private List<IncidentSLA> incidentSLAS=new ArrayList<>();
}
also i have this function :
@Query("SELECT s.name AS slaName,"+
            "COUNT(i.id) AS totalCount,"+
            "SUM(CASE WHEN i.closingDate <= i.dueDate THEN 1 ELSE 0 END) AS countRespectedSLA, "+
            "SUM(CASE WHEN i.closingDate > i.dueDate THEN 1 ELSE 0 END) AS countViolatedSLA "+
            /*"FROM IncidentSLA i JOIN i.typeIncidentSLA tis JOIN tis.sla s JOIN i.incident inc " +*/
            "FROM SLA s LEFT JOIN TypeIncidentSLA tis ON s.id = tis.sla.id LEFT JOIN IncidentSLA i ON tis.id = i.typeIncidentSLA.id "+
            "WHERE i.closingDate IS NOT NULL " +
            "AND i.incident.createdAt BETWEEN :startDate AND :endDate " +
            "AND (:typeIncidentId IS NULL OR i.incident.typeIncident.id = :typeIncidentId ) " +
            "AND (:prestataireId IS NULL OR tis.typeIncident.prestataire.id = :prestataireId ) " +
            "GROUP BY s.name")
    List <Tuple> findSLAConformityCounts(
            @Param("startDate") LocalDateTime startDate,
            @Param("endDate")LocalDateTime endDate,
            @Param("typeIncidentId")Long typeIncidentId,
            @Param("prestataireId") Long prestataireId
    );

that return me the name of sla with count of incident that respected or not the sla 
but i have a broble is that this function only return the sla that are declared b

