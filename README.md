
function shoot() {
  if (!playing || ended || reloading) return;

  if (ammo <= 0) {
    message('SEM MUNIÇÃO! Aperte R');
    return;
  }

  ammo--;
  hud();

  // Efeito de disparo
  muzzle.visible = true;
  muzzleLight.intensity = 8;
  flash.style.opacity = '1';
  gun.position.z = 0.12;

  setTimeout(() => {
    muzzle.visible = false;
    muzzleLight.intensity = 0;
    flash.style.opacity = '0';
    gun.position.z = 0;
  }, 75);

  // Detecta o zumbi no centro da mira
  raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
  raycaster.far = 100;

  const parts = [];

  zombies.forEach(z => {
    z.root.traverse(part => {
      if (part.isMesh) {
        part.userData.zombie = z;
        parts.push(part);
      }
    });
  });

  const hits = raycaster.intersectObjects(parts, false);

  const start = new THREE.Vector3();
  camera.getWorldPosition(start);

  if (hits.length > 0) {
    const hit = hits[0];
    const z = hit.object.userData.zombie;

    // Efeitos visuais da bala
    lineEffect(start, hit.point.clone(), 0xffdf8a, 0.1);
    sparks(hit.point.clone());

    if (z && zombies.includes(z)) {
      z.hp = (z.hp ?? 2) - 1;

      if (z.hp <= 0) {
        scene.remove(z.root);
        zombies.splice(zombies.indexOf(z), 1);
        score += 100;
        message('ZUMBI ELIMINADO! +100');
        if (zombies.length === 0) finish(true);
      } else {
        message('ACERTOU! FALTA 1 TIRO');
      }
    }
  } else {
    const direction = new THREE.Vector3();
    camera.getWorldDirection(direction);
    lineEffect(
      start,
      start.clone().add(direction.multiplyScalar(80)),
      0xffdf8a,
      0.1
    );
  }

  hud();
}
