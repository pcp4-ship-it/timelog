  html+='</div>';
  documento.obterElementoPorID('conteúdo do editor').HTML interno=html;
  documento.obterElementoPorID('editor-modal').lista de classes.adicionar('abrir');
}
assíncrono função aplicarSubstituir(campo,k){
  constante pfx=campo==='cliente'?'c':'e';
  constante velhoEl=documento.obterElementoPorID(`o${pfx}-${k}`), novoEl=documento.obterElementoPorID(`n${pfx}-${k}`);
  constante velhoVal=velhoEl?velhoEl.valor:'', novoValor=novoEl?novoEl.valor.aparar():'';
  se(!novoValor){brinde('⚠ O novo nome não pode ser vazio.');retornar;}
  se(novoValor===velhoVal){brinde('⚠ O nome é igual ao atual.');retornar;}
  setSyncStatus('carregando','Salvando...');
  tentar {
    constante dados=obterCache();
    dados.para cada(r=>{
      se(campo==='funcionário') r.funcionários=(r.funcionários||[r.nome||'']).mapa(n=>n===velhoVal?novoValor:n);
      se(campo==='cliente') r.entradas.para cada(e=>{ se(e.cliente===velhoVal) e.cliente=novoValor; });
    });
    aguardar binPut(dados);
    setSyncStatus('OK','Online ✓');
    brinde('✓ Substituído com sucesso!','OK');
    abrirEditor(); renderReport();
  } pegar { setSyncStatus('errar','Erro'); brinde('⚠ Erro ao salvar.'); }
}
função fecharEditor(){ documento.obterElementoPorID('editor-modal').lista de classes.remover('abrir'); }
// ════════════════════════════════════════
// EXPORTAR CSV
// ════════════════════════════════════════
assíncrono função exportarCSV(){
  constante dados=aguardar binGet();
  se(!dados.comprimento){brinde('⚠ Nenhum dado para exportar.');retornar;}
  constante linhas=[['Dados','Colaborador','Início','Fim','Duração (min)','Atividade','Cliente/Setor']];
  dados.para cada(r=>{
    constante funcionários=r.funcionários||[r.nome||''];
    r.entradas.para cada(e=>{
      constante durante=t2m(e.fim)-t2m(e.começar);
      se(durante<=0)retornar;
      funcionários.para cada(empregado=>{
        linhas.empurrar([r.data, empregado, e.começar, e.fim, durante,
          e.atividade.inclui(',')?`"${e.atividade.substituir(/"/g,'""')}"`:e.atividade,
          e.cliente||''
        ]);
      });
    });
  });
  constante arquivo CSV=linhas.mapa(r=>r.juntar(',')).juntar('\r\n');
  constante bolha=novo Mancha(['\uFEFF'+arquivo CSV],{tipo:'text/csv;charset=utf-8;'});
  Objeto.atribuir(documento.criarElemento('um'),{href:URL.criarURLDeObjeto(bolha),download:`timelog_${novo Data().paraISOString().fatiar(0,10)}.csv`}).clique();
  brinde('✓ CSV exportado!','OK');
}
// ════════════════════════════════════════
// LIMPAR TUDO
// ════════════════════════════════════════
assíncrono função limparTudo(){
  se(!confirmar('Apagar TODOS os registros permanentemente?'))retornar;
  setSyncStatus('carregando','Limpando...');
  tentar {
    aguardar binPut([]);
    teclas selecionadas.claro();
    setSyncStatus('OK','Online ✓');
    renderReport();
    brinde('Dados apagados.','OK');
  } pegar { brinde('⚠ Erro ao limpar.'); }
}
// ════════════════════════════════════════
// IMPORTAR DADOS HISTÓRICOS
// ════════════════════════════════════════
assíncrono função importarDadosHistóricos(){
  constante botão=documento.obterElementoPorID('btn-import');
  botão.desabilitado=verdadeiro; botão.Conteúdo do texto='Importando...';
  setSyncStatus('carregando','Importando...');
  tentar {
    constante existente=aguardar binGet();
    constante IDs existentes=novo Definir(existente.mapa(r=>r.eu ia));
    constante para adicionar=HISTÓRICO.filtro(r=>!IDs existentes.tem(r.eu ia));
    se(!para adicionar.comprimento){
      brinde('Todos os registros históricos já foram importados.','OK');
      botão.estilo.mostrar='nenhum'; retornar;
    }
    constante fundido=[...existente,...para adicionar];
    aguardar binPut(fundido);
    setSyncStatus('OK','Online ✓');
    brinde(`✓${para adicionar.comprimento}registro(s) importado(s)!`,'OK');
    botão.estilo.mostrar='nenhum';
    renderReport();
  } pegar {
    setSyncStatus('errar','Erro');
    brinde('⚠ Erro ao importar. Tente novamente.');
    botão.desabilitado=falso; botão.Conteúdo do texto='📥 Importar histórico';
  }
}
// ════════════════════════════════════════
// BRINDE
// ════════════════════════════════════════
função brinde(mensagem,tipo='avisar'){
  constante t=documento.obterElementoPorID('brinde');
  t.Conteúdo do texto=mensagem; t.estilo.fundo=tipo==='OK'?'#2a9d63':'#c8832f';
  t.lista de classes.adicionar('sobre'); setTimeout(()=>t.lista de classes.remover('sobre'),3200);
}
// ════════════════════════════════════════
// INICIALIZAÇÃO
// ════════════════════════════════════════
(função inicializar(){
  constante hoje=novo Data().paraISOString().fatiar(0,10);
  documento.obterElementoPorID('data-funcional').valor=hoje;
  constante de=novo Data(); de.definirData(de.obterData()-30);
  documento.obterElementoPorID('f-de').valor=de.paraISOString().fatiar(0,10);
  documento.obterElementoPorID('f-para').valor=hoje;
  adicionarEntrada();
  setupChipAC();
  checkSync();
  // Exibir botão de importação se os dados históricos ainda não tiverem sido importados
  constante cache=obterCache();
  constante tem histórico=HISTÓRICO.alguns(r=>cache.encontrar(c=>c.eu ia===r.eu ia));
  se(!tem histórico){
    constante botão=documento.obterElementoPorID('btn-import');
    botão.estilo.mostrar='flexível';
    botão.Conteúdo do texto=`📥 Importar histórico (${HISTÓRICO.comprimento}registros)`;
  }
})();
</roteiro>
</corpo>
</html>
